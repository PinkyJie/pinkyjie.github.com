title: 用 TravisCI 来做持续集成
categories:

- 工具相关

tags:

- 持续集成
- TravisCI
- SauceLabs
- SauceConnect
- yargs
- angular1-webpack-starter

date: 2016-02-27 14:35:57

---

持续集成的好处自不用说，一个是省了手动 build 部署的繁琐，二是每一个提交都有自动跑测试保证质量。平时在公司 Jenkins 用的比较多，开源世界里同样也有能和 Jenkins 比肩的好工具：[TravisCI](https://travis-ci.org/)。TravisCI 可以和 Github 无缝集成，每次 push 都可以触发相应的操作，跑测试、自动部署都不在话下。这篇文章就记录一下我在[angular1-webpack-starter](https://github.com/PinkyJie/angular1-webpack-starter)项目上使用 TravisCI 的经验，最终实现的效果就是：提交一个 commit 后，自动跑单元测试，然后 build 项目，然后将测试覆盖率报告和站点部署到 Github Pages 上去，最后跑 E2E 测试。

<!--more-->

### 基本配置

TravisCI 的上手非常简单，只需三步：

1. 用 Github 账号登录 TravisCI，你的所有 Repo 都会列出来，选择激活你想要 build 的 Repo。
2. 在 Github 的 Repo 里添加`.travis.yml`配置文件。
3. Push 代码到 Github，触发自动 build。

这里要说的就是`.travis.yml`这个配置文件了（有人可能要吐槽了：哦 NO，又一个配置文件，前端的配置文件实在太多了！没办法啊！(\*@ο@\*)），`.travis.yml`的基本格式如下：

```yaml
branches:
  only:
    - master
language: node_js
node_js:
  - stable
script:
  - xxx
after_success:
  - xxx
env:
  global:
    - xxx: xxx
```

基本上不需要解释就可以明白大致的意思，来自 Python 家族的 yml 格式可以说是配置文件中格式最简单，最易读的了。

- `branches`：指定要 build 的是哪个分支，支持 build 多个分支。
- `language`：指定使用的语言，前端项目的 build 过程基本都是 node 啦。
- `node_js`：指定要使用的 node 版本，可以同时指定多个版本，触发多个 build。
- `script`：系统自动运行完`npm install`后会执行这里指定的命令，简单的功能可以直接写一个 npm run script，复杂的逻辑也可以指定执行一个 bash 脚本。
- `after_success`：这里是`script`部分跑成功以后才会执行的命令，这里的命令成功与失败不会影响整个 Travis 任务。
- `env`：这个字段用来定义环境变量，一些需要加密的信息也可以放在这里。

当然除此之外，Travis 还支持很多的配置，可以查看[它的文档](https://docs.travis-ci.com/user/customizing-the-build/#The-Build-Lifecycle)来认识下 build 过程的整个生命周期。

### 跑单元测试并上传覆盖率报告

跑测试是自动化部署的第一步，如果单元测试都跑不过的话也不用 build 项目了。跑单元测试的方法不用多说，配好了 karma 以后，运行一个`npm test`就好。这里简单讲讲其中的两个问题：

1. 识别 Travis 的环境。
2. 生成覆盖率报告并上传。

#### 识别 Travis 的环境

为什么需要专门识别 Travis 的环境呢？由于测试是在 Travis 的云主机上跑的，本地跑测试一般都是直接启一个 Chrome 浏览器，但 Travis 上显然没有 Chrome 浏览器（据说有 Firefox），所以我们需要 headless 的浏览器 PhantomJS 来跑测试。虽然 Travis 自己有环境变量可以帮助我们得知脚本是否运行在 Travis 上，但我推荐的方法还是直接给 npm script 传参数比较好，这样方便以后我们移植到其它 CI 平台。我们要达到的最终目的是：运行`npm test`则用 Chrome 来跑，运行`npm test -- --ci`则用 PhantomJS 来跑。注意这里中间的`--`，根据[官方文档](https://docs.npmjs.com/cli/run-script)的解释，npm 需要这个标记来获知 npm 自己的 options 已经结束，npm 会把中间的`--`之后的部分都传给你自己的脚本。我们的`npm test`在 package.json 中是这样配置的：`karma start ./karma.config.js`，所以我们需要在 karma.config.js 中处理传入的`--ci`参数。不要看 karma.config.js 只是一个配置文件，其实里面可以随便放任何的 nodejs 逻辑，只要保证这个文件最后 export 出一个函数即可：`moudule.exports = (config) => {config.set({...})}`。说到处理参数，nodejs 中有非常好用的工具：[yargs](https://www.npmjs.com/package/yargs)，它支持多种处理命令行参数的方式，使用非常简单。

```javascript
const args = require("yargs").argv;
const browser = args.watch || args.ci ? "PhantomJS" : "Chrome";
const files =
  browser === "PhantomJS"
    ? ["node_modules/babel-core/browser-polyfill.js", unitTestEntry]
    : [unitTestEntry];
module.exports = config => {
  config.set({
    files,
    browsers: [browser]
    // ....
  });
};
```

可以看到，只要将这个包 require 进来，处理参数起来真是得心应手。第 2 行中我们指定如果带有`--watch`或`--ci`参数，则使用 Chrome 浏览器（这里的 watch 模式用于：在开发时代码一更新就自动跑测试，由于需要频繁跑测试，用 PhantomJS 的话跑的更快一些）。另外，我们采用了 ES6，那么在使用 PhantomJS 的时候就需要额外包含 Babel 的 polyfill（第 4 行）。

#### 生成报告并上传

生成覆盖率测试报告自然是要用 karma-coverage 了，只需要在 karma 的配置中配置`coverageReproter`即可。

```javascript
reporters: ['mocha', 'coverage'],
coverageReporter: {
    dir: 'source/test/unit/results/coverage',
    reporters: [
        {type: 'lcov', subdir: '.'},
        {type: 'text-summary'}
    ]
},
```

上面的 lcov 就用来生成覆盖率报告，运行`npm test`以后，就会在指定的路径（dir 配置）里生成覆盖率报告：以网页的形式展示所有文件的覆盖率。如下图：

![图3](/images/continuous-integration-with-travis-ci-3.png)

这里要重点说的是上传这个报告，一方面需要将这个报告也部署到 Github Pages 上方便查看，另一方面还可以将报告上传到“特殊”的网站。前者我们将在下一节详细讲，这里主要说这个“特殊”的网站：[CodeCov](https://codecov.io/)。看名字就知道它是一个专门服务于代码覆盖率的工具，将报告上传后不仅可以在上面方便的查看，关键是可以得到下面这个东西：

[![Codecov](https://img.shields.io/codecov/c/github/PinkyJie/angular1-webpack-starter.svg?style=flat-square)](https://codecov.io/github/PinkyJie/angular1-webpack-starter)

哈哈，把它放在项目的 readme 里是不是特别有范（对类似的 badge 服务感兴趣的可以戳[Shields](http://shields.io/)，里面有各种各样包罗万象的 badge）。上传也是非常的简单，首先安装 CodeCov 提供的[node 工具](https://www.npmjs.com/package/codecov.io)，然后一条 npm script 搞定：`"codecov": "cat ./source/test/unit/results/coverage/lcov.info | codecov",`。这样在`.travis.yml`就可以直接调用`npm run codecov`来上传报告啦。

### 部署到 Github Pages

Travis 自身支持很多[deploy 的服务](https://docs.travis-ci.com/user/deployment/)，但由于[我的项目](https://github.com/PinkyJie/angular1-webpack-starter)是一个纯前端的项目，并不需要后台，所以我选择直接把它部署到 Github Pages 上。Github Pages 其实就相当于一个静态的 server，其部署流程其实就是把 build 后的页面 push 到项目的`gh-pages`分支下即可。要上传我们也要搞定两个问题：

1. 项目中的路由使用 HTML5 模式，需配合 webpack-dev-server 的[historyApiFallback 配置](https://webpack.github.io/docs/webpack-dev-server.html#the-historyapifallback-option)，而 Github Pages 上是纯静态 server，我们需要使用`/#`这种路由模式。
2. `git push`到 Github 是需要权限的，你不可能把自己的私钥上传到 Travis 上去。

#### build 前更改代码

先来看第一个问题，解决这个问题的关键就是，在 Travis 开始 build 之前，我们需要更改代码里的`this.$locationProvider.html5Mode(true);`这一句，将 true 改为 false 即可。那怎么优雅的做这个更改呢？答案是使用 git 的 patch 功能！我们可以先在本地做这个更改，然后运行`git diff > gh-pages-patch.diff`这样，就会得到这样内容的文件：

```bash
diff --git a/source/app/components/_common/services/router-helper.provider.js b/source/app/components/_common/services/router-helper.provider.js
index 426cf72..5319d32 100644
--- a/source/app/components/_common/services/router-helper.provider.js
+++ b/source/app/components/_common/services/router-helper.provider.js
@@ -101,7 +101,7 @@ class RouterHelperProvider {
             mainTitle: '',
             resolveAlways: {}
         };
-        this.$locationProvider.html5Mode(true);
+        this.$locationProvider.html5Mode(false);
     }
```

其实就是将 diff 的内容保存到一个文件，然后这个 patch 文件我们就可以在需要的时候将它应用回去。在 Travis 上 build 之前，我们可以直接运行`git apply ./gh-pages-patch.diff`，这样这行代码的变动就可以应用到 Travis 拿到的代码上，然后再开始 build 就可以保证得到的最终网页是正确的了。

#### Github 的权限问题

关于 Github 的权限问题，其实除了使用私钥外，Github 也是允许使用 Access Token 来 push 代码的，我们就是需要利用这一点来将 build 后的代码 push 回 gh-pages 分支。首先访问 Github 设置里的[Personal access tokens](https://github.com/settings/tokens)，选择“Generate new token”来生成新的 token，给 token 起一个名字然后只选择 repo 权限即可，添加成功后会看到类似下面的页面：

![图1](/assets/images/continuous-integration-with-travis-ci-1.png)

这里必须将 token 保存到你本地，因为 token 只会显示一次，一旦你离开这个页面，再也看不到这个 token 了。有了这个 token，我们就可以使用下面的命令来 push 代码了：

```bash
git push "https://xxx@github.com/PinkyJie/angular1-webpack-starter.git" master:gh-pages
```

其中的 xxx 就是你的 token，而后面是 Github 项目的地址。当然，因为`.travis.yml`文件是公开的，我们不可能就直接把这个 token 明文写在里面，别人有了 token 就相当于有了你 Github 所有 Repo 的 push 权限（这也是为什么 token 在生成后只显示一次）。贴心的 Travis 早就想到了这一点，它提供[对“键值对”进行加密的功能](https://docs.travis-ci.com/user/encryption-keys/#Usage)。首先我们需要安装 Travis 提供的加密工具：`gem install travis`，然后在你的项目目录下（其他目录需要手动指定`-r owner/project`）运行`travis encrypt GITHUB_TOKEN=xxx`就会得到类似`secure: "..."`的输出，将这段输出放在`.travis.yml`中的环境变量块下面即可：

```yaml
env:
  global:
    - GitHub_REF: github.com/PinkyJie/angular1-webpack-starter.git
    - secure: "..."
```

注意，这里的加密是把 key 和 value 都加密了，`.travis.yml`里允许出现多个 secure 的定义，它们都将作为环境变量供你的脚本调用，在脚本中可以使用`${GitHub_TOKEN}`和`${GitHub_REF}`引用上面定义的变量。最后我们还可以给 Travis 的 push 提供一个假的身份，这样在 Github 上我们一看到某个 commit 就知道这是 Travis 里 push 上来的，最终的脚本如下：

```bash
cd build
git init
git config user.name "Travis CI"
git config user.email "travis@pinkyjie.com"
git add .
git commit -m "Deploy at `date +"%Y-%m-%d %H:%M"`"
git push --force --quiet "https://${GitHub_TOKEN}@${GitHub_REF}" master:gh-pages > /dev/null 2>&1
```

可以看到第 3-4 行，我们给 Travis 配了一个名字和假的 Email，第 6 行我们给 commit 配了一个包含时间戳的 message，这样就很容易从 Github 上知道这次部署发生在几点了。

除了上传 build 后的代码，我们还可以将前面提到的单元测试覆盖率报告也上传到 Github Pages 上去，只需要在 build 完成后将生成的 coverage 目录复制到 build 目录即可。有一点值得注意的就是 Github Pages 有一个设置就是：下划线`_`开头的文件夹名在 web 上无法访问。如果你的代码结构中有下划线命名的文件夹，则单元测试报告里也会有同名的文件夹，我们可以将其重命名：

```bash
mv ./build/coverage/app/components/_common ./build/coverage/app/components/common
sed -i 's/href=\"app\/components\/_common/href=\"app\/components\/common/g' ./build/coverage/index.html
```

这里先对文件夹进行重命名，再对文件内容里涉及到的相应超链接进行更新。

### 用 Sauce Labs 跑 E2E 测试

部署好了自然是要在部署的站点上来一个 E2E 测试了，这也是为什么我们要把 E2E 测试放在`after_success`这个阶段，因为需要等`script`阶段的部署完成才可以。前端的一个重要的问题就是浏览器兼容性问题，所以我们当然希望 E2E 测试能够覆盖越多的浏览器越好，比如 IE、Chrome、Safari，甚至 iOS 和 Android 上的浏览器。这个时候就需要[Sauce Labs](https://saucelabs.com/)出场了，它就相当于一个云端的浏览器仓库，包含各种设备各种系统上的不同浏览器，可以通过配置文件让你的测试跑在它上面。这么好的服务自然不是免费的，好消息是：对于开源项目，它是完全免费的，虽然只允许同时运行 5 个浏览器实例，但对于我们来说已经足够了。protractor 的配置文件本身就是支持 Sauce Labs 的，只需要在配置文件中添加`sauceUser`和`sauceKey`即可，分别是你注册 Sauce Labs 时的用户名以及自动生成的 Access Key。跟 karma 的配置文件类似，我们也需要在 protractor 的配置文件中处理传入参数。看下面这个例子：

```javascript
if (args.ci) {
  // run by sauce lab
  config.baseUrl = "http://pinkyjie.com/angular1-webpack-starter/#";
  config.sauceUser = "xxx";
  config.sauceKey = "xxx";
  config.multiCapabilities = [
    {
      name: `Chrome (build-${args.buildId})`,
      build: `${args.buildId}`,
      browserName: "chrome",
      platform: "Windows 7"
    },
    {
      name: `Safari (build-${args.buildId})`,
      build: `${args.buildId}`,
      browserName: "safari",
      platform: "OS X 10.11"
    },
    {
      name: `IE (build-${args.buildId})`,
      build: `${args.buildId}`,
      browserName: "internet explorer",
      platform: "Windows 7",
      version: "11.0"
    },
    {
      name: `iOS (build-${args.buildId})`,
      build: `${args.buildId}`,
      browserName: "Safari",
      appiumVersion: "1.4.16",
      deviceName: "iPhone 5s",
      deviceOrientation: "portrait",
      platformVersion: "9.1",
      platformName: "iOS"
    },
    {
      name: `Android (build-${args.buildId})`,
      build: `${args.buildId}`,
      browserName: "Browser",
      appiumVersion: "1.4.16",
      deviceName: "Android Emulator",
      deviceOrientation: "portrait",
      platformVersion: "4.4",
      platformName: "Android"
    }
  ];
} else {
  // local run
  config.baseUrl = "http://localhost:8080";
  config.multiCapabilities = [
    {
      browserName: "chrome"
    },
    {
      browserName: "chrome",
      chromeOptions: {
        args: ["--window-size=375,627"]
      }
    }
  ];
}
```

在配置文件中，我们判断如果传入了`--ci`则添加`sauceUser`和`sauceKey`，否则就不加。另外，有 ci 的时候我们需要传入不同的`baseUrl`，没 ci 则认为是在本地跑测试。两者都使用了`multiCapabilities`，ci 上跑的时候指定了我们要测试的 5 个浏览器，本地则是开了两个 Chrome，一个全屏一个模拟手机分辨率来测试。我们重点来看看使用 Sauce Labs 时的 5 个浏览器配置，它们分别是 Chrome、Safari、IE、Android 浏览器，和 iOS 上的 Safari（Android 和 iOS 的测试使用了[Appium](http://appium.io/)），当然 Sauce Labs 还支持很多的浏览器，可以通过[其文档](https://wiki.saucelabs.com/display/DOCS/Platform+Configurator#/)来生成指定浏览器的配置。上面的例子中除了浏览器的系统、名字和版本外，我们还指定了`name`和`build`，这也是 Sauce Labs 特有的配置，这两个配置可以让 Sauce Labs 更好的显示和组织属于同一个 build 的测试，如下图：

![图2](/assets/images/continuous-integration-with-travis-ci-2.png)

属于同一个 build 的所有 E2E 测试被组织到了一起。这里传入的 buildId 则是 Travis 每次 build 自动生成的 ID，我们可以通过环境变量获取：`npm run e2e -- --ci --build-id=$TRAVIS_BUILD_NUMBER`。

#### 本地的站点也能测

有这样一种场景，如果你的网站架在本地的话（比如 localhost），外网是无法访问的，这样在 Sauce Labs 上的云主机也就无法访问你的网站，自然也无法跑测试的。为了解决这个需求，Sauce Labs 还提供了一个有用的功能：[Sauce Connect](https://wiki.saucelabs.com/display/DOCS/Using+Sauce+Connect+for+Testing+Behind+the+Firewall+or+on+Localhost)。它会在本地开启一个 tunnel，用来接收 protractor 的命令，然后传给 Sauce Labs 云主机上的浏览器。幸运的是 Travis 也支持这个功能，只需要在`.travis.yml`中添加指定的配置即可：

```yaml
addons:
  sauce_connect:
    username: "xxx"
    access_key: "xxx"
  apt:
    sources:
      - ubuntu-toolchain-r-test
    packages:
      - gcc-4.8
      - g++-4.8
```

然后在 protractor 的配置文件中，给每个浏览器的配置添加`'tunnel-identifier': args.jobId`即可，而这个 jobId 也是 Travis 每次 build 自动生成的，也可以通过环境变量获取：`npm run e2e -- --ci --build-id=$TRAVIS_BUILD_NUMBER --job-id=$TRAVIS_JOB_NUMBER`。这样，你就可以在 Travis 的机器上启一个本地的网站，然后通过 Sauce Connect 来测试它了。

#### Timeout 的问题

这是我自己碰到的一个问题，E2E 测试在本地跑的好好的，但放在 Sauce Labs 上总是提示超时。protractor 的文档专门[有一个章节](http://www.protractortest.org/#/timeouts)来讲超时的问题。简单的说，如果出现超时，可以从两方面入手，一个是增加 jasmine 的默认超时时间，即在 protractor 的配置中加入：

```javascript
jasmineNodeOpts: {
  defaultTimeoutInterval: 600 * 1000; // 毫秒
}
```

另一方面，Sauce Labs 也有 3 个参数来控制超时，可以在 protractor 的配置文件中给每个浏览器的配置中加入：

```javascript
// 秒
maxDuration: 3600,
commandTimeout: 600,
idleTimeout: 1000
```

> 就我个人的经验，目前项目的 E2E 测试在本地跑完全没问题，在 Sauce Labs 上仍然会报错，目前仍在研究中（[issue 9](https://github.com/PinkyJie/angular1-webpack-starter/issues/9)）

最后，完整的实现可以戳这里：

- [.travis.yml](https://github.com/PinkyJie/angular1-webpack-starter/blob/master/.travis.yml)
- [script 部分的脚本](https://github.com/PinkyJie/angular1-webpack-starter/blob/master/publish-to-gh-pages.sh)
- [karma.config.js](https://github.com/PinkyJie/angular1-webpack-starter/blob/master/karma.config.js)
- [protractor.config.js](https://github.com/PinkyJie/angular1-webpack-starter/blob/master/protractor.config.js)
