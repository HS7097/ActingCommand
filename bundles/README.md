# bundles/

English · [简体中文](#简体中文)

This directory carries one resource bundle per game, exactly as its resource repository published it: `<game>-bundle-<sha7>.zip` and its `.sha256` sidecar, byte-identical to the release assets. While the `RESOURCES_READ_TOKEN` secret is not configured, the `publish-artifacts` workflow attaches these bundles to every release instead of downloading them from the resource repositories.

When a resource repository publishes a new bundle, its two files here are replaced, not accumulated: there is always exactly one bundle per game.

The `.sha256` sidecar is the integrity statement: the workflow verifies the zip against it (`sha256sum -c`) and fails if it is missing or does not match.

## 简体中文

本目录按游戏各放一个资源包，与资源仓发布的原样一致（与发布资产逐字节相同）：`<game>-bundle-<sha7>.zip` 及其 `.sha256` 校验文件。在未配置 `RESOURCES_READ_TOKEN` 密钥时，`publish-artifacts` 工作流把这里的资源包附进每个发布，而不是从资源仓下载。

资源仓发布新的资源包时，这里的两个文件被替换而不是累积：每个游戏始终只有一个资源包。

`.sha256` 校验文件就是完整性声明：工作流用它校验 zip（`sha256sum -c`），缺失或不符即失败。
