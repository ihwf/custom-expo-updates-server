# 注意：需要更新 `.env.local` 中的 `HOSTNAME` 为当前服务地址。用于下载更新包和资源。

# 自定义 Expo Updates 服务器与客户端

本仓库包含一个实现了 [Expo Updates 协议规范](https://docs.expo.dev/technical-specs/expo-updates-0) 的服务器和客户端。

> [!IMPORTANT]  
> 本仓库旨在提供一个协议实现的基本示例。它不保证完整、稳定或具备足够的性能来作为 `expo-updates` 的完整后端使用。Expo 不会对该仓库的自定义服务器实现提供技术支持。有关 `expo-updates` 客户端库的问题（与服务器无关的部分）可以在 https://github.com/expo/expo/issues/new/choose 提交。任何新增功能的 Pull Request 可能会被关闭；如有需要，请自行 fork 仓库并在此基础上添加新功能。

## 背景

Expo 提供了一套名为 EAS（Expo 应用服务）的服务，其中之一是 EAS Update，它可以使用 [`expo-updates`](https://github.com/expo/expo/tree/main/packages/expo-updates) 库为主机和分发 Expo 应用程序的更新提供支持。

在某些情况下，可能需要对如何向应用程序发送更新进行更多控制，一种选择是实现一个符合规范的自定义更新服务器，以便提供更新清单和资源。本仓库包含了一个符合规范的服务器实现示例以及一个配置为使用该示例服务器的客户端应用程序。

## 开始使用

### 更新概述

要理解本仓库，了解一些关于更新的术语非常重要：

- **运行时版本**：类型：字符串。运行时版本指定了您的应用程序正在运行的基础原生代码的版本。当更新依赖于新的或更改的原生代码时（例如，当您更新 Expo SDK 或将任何原生模块添加到应用程序中），您需要更新更新的运行时版本。如果更新依赖于用户未运行的原生代码，则未能更新更新的运行时版本会导致最终用户的设备崩溃。
- **平台**：类型："ios" 或 "android"。指定要提供更新的目标平台。
- **清单**：在协议中有描述。清单是一个对象，描述了 Expo 应用程序需要知道以加载更新的资源和其他详细信息。

### 如何工作 `expo-update-server`

创建更新的流程如下：

1. 配置并构建应用程序的一个“发布”版本，然后在模拟器上运行或将应用部署到应用商店。
2. 在本地运行项目，进行修改，然后将应用程序导出为更新。
3. 在服务器仓库中，我们将 #2 中制作的更新复制到 **expo-update-server/updates** 目录下，位于相应的运行时版本子目录中。
4. 在“发布”应用程序中，强制关闭并重新打开应用程序，以从自定义更新服务器请求更新。服务器将返回与请求的平台和运行时版本匹配的清单。
5. 一旦“发布”应用程序接收到清单，它将对每个资源发出请求，这些资源也将由此服务器提供。
6. 一旦应用程序从服务器获取所有所需的资源，它将加载更新。

## 设置

注意：应用程序配置为从 http://localhost:3000 运行的服务器加载更新。如果您希望从不同的基础 URL 加载它们（例如，在 Android 模拟器中）：

1. 更新服务器中的 `.env.local`。
2. 更新 `app.json` 中的 `updates.url` 并重新运行以下步骤中的构建步骤。

### 创建一个“发布”应用程序

配置为服务器的示例 Expo 项目位于 **/expo-updates-client** 中。

#### iOS

运行 `yarn` 和 `yarn ios --configuration Release`。

#### Android

运行 `yarn` 然后运行 `yarn android --variant release`。

### 进行修改

让我们对 /expo-updates-client 中的项目进行一些修改，我们希望将其作为通过我们的自定义服务器推送到“发布”应用程序的空中更新。进入 **/expo-updates-client**，然后修改 **App.js**。

完成满意的修改后，在 **/expo-updates-server** 中运行 `yarn expo-publish`。在幕后，此脚本会在客户端运行 `npx expo export`，将导出的应用程序复制到服务器，并同时将 Expo 配置复制到服务器。

### 发送更新

现在我们可以运行更新服务器了。在本仓库的服务器文件夹中运行 `yarn dev` 来启动服务器。

在运行“发布”版本应用程序的模拟器中，强制关闭应用程序并重新打开。它应该会向 `/api/manifest` 发起请求，然后向 `/api/assets` 发起请求。应用程序加载后，它应该显示您本地所做的任何更改。

## 关于此服务器

此服务器是使用 NextJS 创建的。您可以在 **pages/api/manifest.js** 和 **pages/api/assets.js** 中找到 API 端点。

代码签名密钥和证书是使用 https://github.com/expo/code-signing-certificates 生成的。