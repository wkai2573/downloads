# downloads

wkai2573 作品的下載頁，只放發行版。

這裡只收原始碼在私有 repo 的作品，因為私有 repo 的 Release 頁外人會 404，所以把可下載的版本發布到這個公開 repo，讓任何人都能直接下載。

作品 repo 如果是公開的，不需要發布到這裡，直接用自己 repo 的 GitHub Release 即可。

## 作品

| 作品 | 下載頁 | 直接下載 |
| --- | --- | --- |
| fracture-lab | [Release](https://github.com/wkai2573/downloads/releases/tag/fracture-lab) | [fracture-lab-windows.zip](https://github.com/wkai2573/downloads/releases/download/fracture-lab/fracture-lab-windows.zip) |

## 規則

- 一個作品對應一個 Release，tag 名稱就是作品名，例如 `fracture-lab`。
- 新版直接覆蓋舊版，不保留歷史版本，所以下載連結永遠不變。
- 附件檔名不含版本號，例如 `fracture-lab-windows.zip`。版本號寫在 Release 的標題與說明。
- 下載頁網址格式為 `https://github.com/wkai2573/downloads/releases/tag/<作品名>`。
- 直接下載網址格式為 `https://github.com/wkai2573/downloads/releases/download/<作品名>/<檔名>`。
- 不要用 `releases/latest`，它會指向整個 repo 最近建立的 Release，有多個作品時會指到別人。
- 不要把檔案提交進 git 目錄，只放在 Release 附件。

## 新作品要怎麼發布到這裡

發布由各作品 repo 的 GitHub Actions 負責，不需要手動操作這個 repo。

### 一次性設定

1. 準備發布用的 fine-grained PAT，全部作品共用同一個：
   - Repository access 只選 `wkai2573/downloads`。
   - Permissions 的 Contents 選 Read and write，其他都不勾。
   - 有效期最長一年，到期前 GitHub 會寄信提醒。
2. 到作品 repo 的 Settings → Secrets and variables → Actions，新增 secret，名稱為 `RELEASES_PUSH_TOKEN`，值貼上這個 token。

### 在作品 repo 加入發布步驟

在作品的 workflow 裡，建好要發布的檔案後，加上這一步。觸發方式建議用推送 `v*` tag，`APP_NAME` 與 `ASSET` 依作品調整：

```yaml
env:
  DOWNLOADS_REPO: wkai2573/downloads
  APP_NAME: my-app
  ASSET: my-app-windows.zip

steps:
  # ... 建置並產生 ${ASSET}

  - name: Publish to downloads repo
    if: github.ref_type == 'tag'
    env:
      GH_TOKEN: ${{ secrets.RELEASES_PUSH_TOKEN }}
      VERSION: ${{ github.ref_name }}
    run: |
      if [ -z "${GH_TOKEN}" ]; then
        echo "::error::缺少 secret RELEASES_PUSH_TOKEN"
        exit 1
      fi
      echo "目前版本：${VERSION}" > notes.md
      if gh release view "${APP_NAME}" --repo "${DOWNLOADS_REPO}" >/dev/null 2>&1; then
        gh release upload "${APP_NAME}" "${ASSET}" --repo "${DOWNLOADS_REPO}" --clobber
        gh release edit "${APP_NAME}" --repo "${DOWNLOADS_REPO}" \
          --title "${APP_NAME} ${VERSION}" --notes-file notes.md
      else
        gh release create "${APP_NAME}" "${ASSET}" --repo "${DOWNLOADS_REPO}" \
          --title "${APP_NAME} ${VERSION}" --notes-file notes.md
      fi
```

- 這一步的行為是：Release 不存在就建立，已存在就覆蓋同名附件並更新標題與說明。
- 作品 repo 的 workflow 權限只需要 `contents: read`，發布用的是 token，不是內建的 `GITHUB_TOKEN`。
- 不要用 `--generate-notes`，它會讀這個 repo 的 commit，而這裡沒有作品的提交紀錄。

### 發布新版

在作品 repo 推送新的 tag 即可：

```bash
git tag v1.0.1
git push origin v1.0.1
```

發布完成後，到下載頁確認附件已更新。

### 把下載頁放到別處

hub 首頁這類入口，連結請指向 `https://github.com/wkai2573/downloads/releases/tag/<作品名>`。

## Token 維護

- 到期後所有作品的發布都會失敗，錯誤訊息是授權失敗。
- 重新產生 token 後，要到每一個用到它的作品 repo，更新 `RELEASES_PUSH_TOKEN` 這個 secret。
- 每新增一個發布到這裡的作品，就把作品名加進上面的「作品」表格。
