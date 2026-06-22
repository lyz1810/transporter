# GitHub Actions 上传 iOS IPA 说明

这个 workflow 用来把 uniapp 打包出来的 iOS `.ipa` 上传到 App Store Connect/TestFlight，可以替代手动打开 Transporter 上传。

当前默认上传路径是：

```text
dist/sohobao-1.4.2.ipa
```

注意：GitHub Actions 运行在 GitHub 的 macOS 机器上，不能直接读取你电脑里的 `E:\Program Files\...` 本地路径。正式运行前，需要先满足其中一种方式：

- 把 `sohobao-1.4.2.ipa` 放到仓库的 `dist/sohobao-1.4.2.ipa` 路径里。
- 在 workflow 里增加 uniapp/HBuilderX 打包步骤，让它输出到 `dist/sohobao-1.4.2.ipa`。
- 在 workflow 里增加下载步骤，从 GitHub Release、OSS、COS、S3 等位置下载 `.ipa` 到 `dist/sohobao-1.4.2.ipa`。

## 需要配置的 Apple 信息

App Store Connect API Key 信息已经填进 workflow：

- Issuer ID：`e278c62f-a5f2-4d9e-a72e-554d0b604a6d`
- Key ID：`A8CWSNDUUT`

你还需要在 GitHub 仓库里配置一个 Secret：

- Repository secret `APPSTORE_API_PRIVATE_KEY`：下载的 `AuthKey_A8CWSNDUUT.p8` 文件完整内容。

`.p8` 私钥只能下载一次，请保存好。如果丢失，只能撤销旧 Key 后重新创建。

## App Store Connect 里从哪里拿

1. 打开 App Store Connect：https://appstoreconnect.apple.com/
2. 进入 `用户和访问` / `Users and Access`。
3. 进入 `集成` / `Integrations`。
4. 选择 `App Store Connect API`。
5. 创建 Team API Key，权限建议至少使用可以上传构建版本的角色，例如 `App Manager` 或 `Admin`。
6. 生成后页面会显示 `Key ID`，当前 workflow 已填写为 `A8CWSNDUUT`。
7. 下载 `.p8` 文件，把文件完整文本内容填到 GitHub secret `APPSTORE_API_PRIVATE_KEY`。
8. 页面上还能看到 `Issuer ID`，当前 workflow 已填写为 `e278c62f-a5f2-4d9e-a72e-554d0b604a6d`。

## GitHub 里从哪里设置

在你的 GitHub 仓库页面：

1. 进入 `Settings`。
2. 左侧进入 `Secrets and variables`。
3. 点击 `Actions`。
4. 在 `Secrets` 标签页新增：
   - `APPSTORE_API_PRIVATE_KEY`

`APPSTORE_API_PRIVATE_KEY` 必须放到 Secrets，不要放到 Variables，也不要提交到代码仓库。

本机私钥文件路径是：

```text
E:\Program Files\wx\xwechat_files\yz1810_347d\temp\RWTemp\2026-06\a031f0ad6d09f532e2a98c6d7a85b332\AuthKey_A8CWSNDUUT.p8
```

打开这个 `.p8` 文件，复制从 `-----BEGIN PRIVATE KEY-----` 到 `-----END PRIVATE KEY-----` 的完整内容，粘贴到 GitHub Secret 的 Value 中。

## 如何运行

1. 确认 `.ipa` 在 workflow 运行环境里存在，例如 `dist/sohobao-1.4.2.ipa`。
2. 打开 GitHub 仓库的 `Actions`。
3. 选择 `上传 iOS IPA 到 App Store Connect`。
4. 点击 `Run workflow`。
5. `ipa_path` 默认是 `dist/sohobao-1.4.2.ipa`，如果文件路径不同就改成实际路径。

## 说明

- workflow 使用 `apple-actions/upload-testflight-build` 的 `appstore-api` 后端，不需要安装 Transporter 桌面端。
- 上传成功后，构建版本会先进入 App Store Connect 处理队列，处理完成后才能在 TestFlight 或提交审核页面里看到。
- 加密合规状态建议在 App Store Connect 后台设置。workflow 不再传 `uses-non-exempt-encryption`，避免构建已经设置过该字段时出现 `You cannot update when the value is already set` 的 409 错误。
