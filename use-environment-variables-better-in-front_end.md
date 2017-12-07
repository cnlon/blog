<script type="application/ld+json">
{
  "@type": "Article",
  "headline": "如何更好的管理前端环境变量",
  "name": "use-environment-variables-better-in-front_end",
  "url": "https://lon.im/post/use-environment-variables-better-in-front_end.html",
  "dateCreated": "2017-10-10",
  "dateModified": "2017-12-05",
  "datePublished": "2017-12-05"
}
</script>


本文主要分析使用环境变量管理前端项目时会遇到的问题，并介绍常用工具给出解决方案。

## 如何使用环境变量

在搭建基于 webpack 前端项目时（或任意基于 Node 的项目，本文以 webpack 项目为例），一般需要提供两种运行模式：开发模式和生产模式。通常的做法是，执行命令前设置环境变量 `NODE_ENV` 为 `production`，如执行 `NODE_ENV=production webpack` 命令，然后在 JavaScript 代码中通过 `process.env.NODE_ENV === 'production'` 来判断是生产模式，否则为开发模式。通过区分不同的模式可以执行不同的操作，比如在开发模式下启动开发服务器并代理转发 API，或在生产模式下压缩合并代码等。为了更好的统一前端工程命令，可以将启动开发模式和生产模式的命令分别加入 package.json 文件的 scripts 字段中，以后只需要执行 `npm start` 或 `npm run build` 即可。通过定义环境变量的方式很好的解决了在项目中执行差异操作的需求。如果希望支持成员自定义环境变量，只要在程序中优先使用环境变量中的值即可。比如已经设置端口号优先使用环境变量中的 `PORT` 的值，项目成员开发时执行 `PORT=8080 npm start` 命令就可以自定义端口号为 8080 了。

## 使用环境变量时遇到的问题

上述的解决方案可以适用大部分场景，但却无法解决设置环境变量的跨平台和持久化问题

**跨平台**

如果项目中有使用 Windows 操作系统的成员，在执行 `npm run build` （即 `NODE_ENV=production webpack`）时会失败，原因是 Windows 命令不支持使用这种方式设置环境变量。虽然在 Windows 下也可以根据 build 脚本内容，手动执行 `set NODE_ENV=production webpack`，却破坏了统一前端工程命令的初衷，为此需要引入一个解决跨平台设置环境变量的库。如使用 [cross-env](https://github.com/kentcdodds/cross-env)，只要改写 package.json 中的 build 脚本为 `cross-env NODE_ENV=production webpack` 就可以跨平台工作了。

**持久化**

随着规模的增大，项目自定义环境变量的数量可能越来越多。比如部署后静态资源需要使用 CDN，项目生产模式就需要提供一个环境变量用于支持自定义 webpack 的 publicPath 字段；又比如有的成员并没有把 API 服务器运行在本机，而是运行在虚拟机里或另一台电脑上，项目开发模式就需要提供两个环境变量用于支持自定义 API 服务器地址和端口号……可能有的成员每次开发时必须执行类似这么长的命令：`PORT=8080 API_SERVER=192.168.100.100 API_PORT=9000 npm start`，因此需要一个可以持久化环境变量的工具，比如使用 [dotenv](https://github.com/motdotla/dotenv) 或 [env-cmd](https://github.com/toddbluhm/env-cmd) 。以 env-cmd 为例，只需创建一个 .env.local 文件（不计入版本管理），写入：

```ini
NODE_ENV=development
PORT=8080
API_SERVER=192.168.100.100
API_PORT=9000
```

改写 package.json 中 start 命令（build 命令类似）为 `env-cmd --fallback ./.env.local webpack` 即可解决自定义环境变量过多每次手动输入繁琐的问题。

## 真正好用的环境变量管理工具

管理环境变量有很多工具，下面简单分析一下常用工具 [dotenv](https://github.com/motdotla/dotenv)、[cross-env](https://github.com/kentcdodds/cross-env)  和 [env-cmd](https://github.com/toddbluhm/env-cmd) 的优势与不足：

- dotenv 可以解决跨平台和持久化的问题，但使用场景有限，只适用 node 项目，且和项目代码强耦合，需要在 node 代码运行后手动执行触发
- cross-env 支持在命令行自定义环境变量。问题也非常明显，不能解决大型项目中自定义环境变量的持久化问题
- env-cmd 也可以解决跨平台和持久化的问题，支持定义默认环境变量，不足的是不支持在命令行中自定义环境变量

事实上 NPM 本身也提供了[类似设置项目环境变量的功能](https://docs.npmjs.com/misc/config#per-package-config-settings)。以上述自定义端口号的需求为例，也可以在项目目录下执行 `npm config set project-name:PORT 8080`（project-name 为项目名称），执行 `npm start` 后在代码中可以通过 `process.env.npm_package_config_PORT` 获取到 8080。而且还可以将 [package.json 中 config 字段](https://docs.npmjs.com/files/package.json#config)设置为 `{"PORT": 8000}`，用于指定 `npm_package_config_PORT` 的默认值。使用 NPM 的 config 功能管理环境变量的最大优势是原生支持，放在 package.json config 字段中的默认环境变量也非常方便查看。遗憾是的，变量名前面都会有冗长的 `npm_package_config_`；脚本必须从 package.json 的 scripts 字段中执行（即执行 npm run your_script_name）；还有就是所有项目共用一份配置文件（.npmrc，默认在用户目录下），不方便手动编辑和查看。

因此一个好用的前端环境变量管理工具应该具备以下功能：

- 支持命令行设置环境变量
- 跨平台
- 持久化，最好能够提供一个设置本地环境变量的命令行工具
- 支持设置默认环境变量
- 支持获取 NPM 提供的环境变量（`npm_package_*` 和 `npm_config_*`）

为此又诞生了一个环境变量管理工具：[fuck-env](https://github.com/cnlon/fuck-env)，取义“恶搞环境变量”，支持以上所有功能。

**[fuck-env](https://github.com/cnlon/fuck-env) 安装和使用**

```bash
npm install fuck-env
```

如有一个包含 package.json 和 main.js 两个文件的项目，文件代码如下：

package.json

```json
{
  "name": "fuck-env-demo",
  "config": {
    "PORT": 8000,
    "APP_NAME": "$npm_package_name"
  },
  "scripts": {
    "start": "fuck-env node main.js"
  },
  "dependencies": {
    "fuck-env": "*"
  }
}
```

main.js

```javascript
console.log(process.env.PORT)     // 8080
console.log(process.env.APP_NAME) // fuck-env-demo
```

执行 `fuck-env PORT=8080 npm start` 后，输出“8080”和“fuck-env-demo”，不论是在 Windows 还是 POSIX（macOS、Linux 等）系统中。

如果成员希望本地持久化自定义的端口号，可以新建一个 .env 文件（此文件须加入 .gitignore，不计入版本管理，格式为类 .ini 文件的简单键值对）。

.env

```ini
PORT=8080
```

以后只需执行 `npm start` 即可。此外 [fuck-env](https://github.com/cnlon/fuck-env) 还提供了另一个命令行工具：fuck，用于快速设置本地环境变量。比如，如果成员又希望使用 9000 端口，可以在项目根目录下执行 `fuck set PORT 9000`（需全局安装 fuck-env），此时项目目录下 .env 文件的内容即会变为“PORT=9000”，使用 fuck 命令在环境变量较多时非常方便。

当环境变量过多时，全部放置 package.json 的 config 字段也会显得臃肿。fuck-env 支持统一管理默认环境变量，只需将 config 字段下所有环境变量移至 default.env 文件（计入版本库）中即可。

更多实例请参考[这里](https://github.com/cnlon/fuck-env/tree/master/examples/)

[fuck-env](https://github.com/cnlon/fuck-env) 致力于解决用户管理环境变量时遇到的各种问题，未来会加入更多人性化设计。如果你有任何想法，欢迎[给项目提出宝贵的建议](https://github.com/cnlon/fuck-env/issues)。
