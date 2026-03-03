# security_patch_script_launcher.sh



## Code

```
LAUNCHER_SCRIPT_PATH=$(dirname "$0")
PYTHON_SCRIPT_NAME="$1"
shift

export PYTHONUNBUFFERED=TRUE
export PYTHONPATH="${LAUNCHER_SCRIPT_PATH}/../code_integration:${LAUNCHER_SCRIPT_PATH}/../common:${LAUNCHER_SCRIPT_PATH}"
python3 -m pipenv run python3 ${LAUNCHER_SCRIPT_PATH}/${PYTHON_SCRIPT_NAME} "$@"
```



## Explain

這個腳本是一個 Python 腳本啟動器，用於在 pipenv 虛擬環境中執行 Python 腳本。

**逐行說明：**

**第1行：** `LAUNCHER_SCRIPT_PATH=$(dirname "$0")` - 獲取此啟動器腳本所在的目錄路徑

**第2行：** `PYTHON_SCRIPT_NAME="$1"` - 將第一個參數（要執行的 Python 腳本名稱）存入變數

**第3行：** `shift` - 移除第一個參數，讓剩餘的參數可以傳給 Python 腳本

**第5行：** `export PYTHONUNBUFFERED=TRUE` - 設定 Python 為無緩衝模式，讓輸出立即顯示（對日誌記錄很重要）

**第6行：** `export PYTHONPATH=...` - 設定 Python 模組搜尋路徑，包含三個目錄：

- `code_integration` - 程式碼整合模組
- `common` - 共用模組
- 當前腳本目錄

**第7行：** `python3 -m pipenv run python3 ${LAUNCHER_SCRIPT_PATH}/${PYTHON_SCRIPT_NAME} "$@"` - 使用 pipenv 執行指定的 Python 腳本，並傳遞所有剩餘參數

**使用範例：**

```bash
./security_patch_script_launcher.sh generate_security_patch_by_manifest.py --security_tags "tag1" --output_dir "output"
```



# Python Pipenv 詳細說明
Pipenv 是由 Python 官方推薦的「新一代包管理工具」，整合了 `pip`（包安裝）、`virtualenv`（虛擬環境）的核心功能，並解決了傳統 `requirements.txt` 依賴版本混亂、虛擬環境管理繁瑣的問題。它結合了「虛擬環境隔離」與「依賴鎖定」，讓 Python 專案的依賴管理更規範、可重現。

## 一、核心優勢
1. **自動管理虛擬環境**：無需手動創建/激活虛擬環境，Pipenv 會自動為專案創建獨立的虛擬環境，並管理路徑。
2. **雙文件管理依賴**：
   - `Pipfile`：替代 `requirements.txt`，以 TOML 格式清晰區分「生產依賴」（packages）和「開發依賴」（dev-packages）。
   - `Pipfile.lock`：鎖定所有依賴的精確版本（包括子依賴），確保不同環境安裝的依賴版本完全一致。
3. **解決依賴衝突**：自動解析依賴樹，檢測並提示版本衝突，比手動維護 `requirements.txt` 更可靠。
4. **兼容 pip 命令**：支持大部分 `pip` 語法，學習成本低。
5. **安全檢查**：內建 `pipenv check` 命令，可檢查依賴的安全漏洞。

## 二、安裝 Pipenv
### 1. 基礎安裝（全局）
```bash
# 使用 pip 安裝（建議加 --user 避免權限問題）
pip install --user pipenv

# 若提示命令未找到，需將 Python 用戶目錄加入環境變量
# Linux/macOS：將 ~/.local/bin 加入 PATH
# Windows：將 %APPDATA%\Python\PythonXX\Scripts 加入 PATH
```

### 2. 驗證安裝
```bash
pipenv --version
# 輸出 Pipenv, version x.x.x 即安裝成功
```

## 三、核心使用流程
### 1. 初始化專案（創建 Pipfile + 虛擬環境）
進入你的專案目錄，執行以下命令初始化：
```bash
cd your_project
pipenv install  # 無參數：創建 Pipfile 和空的虛擬環境
```
此時專案目錄會生成：
- `Pipfile`：記錄依賴配置
- `Pipfile.lock`：初始鎖定文件（空）

### 2. 安裝依賴
#### （1）安裝生產依賴（上線環境需要的包）
```bash
# 安裝指定包（如 requests）
pipenv install requests

# 安裝指定版本
pipenv install requests==2.31.0

# 安裝 git 倉庫的包
pipenv install git+https://github.com/psf/requests.git@main#egg=requests
```

#### （2）安裝開發依賴（僅開發環境需要，如 pytest、flake8）
```bash
pipenv install --dev pytest  # --dev 標記為開發依賴
```

#### （3）從現有 requirements.txt 遷移
若已有 `requirements.txt`，可直接導入：
```bash
pipenv install -r requirements.txt
```

### 3. 激活/使用虛擬環境
#### （1）進入虛擬環境的 shell
```bash
pipenv shell  # 激活虛擬環境，命令行前會顯示 (專案名) 前綴
# 此時執行 python/ pip 均使用虛擬環境內的版本
```

#### （2）直接運行命令（無需進入 shell）
```bash
pipenv run python script.py  # 用虛擬環境的 Python 運行腳本
pipenv run pytest  # 用虛擬環境的 pytest 執行測試
```

### 4. 鎖定依賴版本
安裝新包後，建議手動更新 `Pipfile.lock`（也可自動生成）：
```bash
pipenv lock  # 重新生成 Pipfile.lock，鎖定所有依賴的精確版本
```

### 5. 部署環境（安裝鎖定的依賴）
在生產環境中，安裝 `Pipfile.lock` 中鎖定的版本（保證一致性）：
```bash
pipenv install --deploy  # --deploy 檢查是否與 Pipfile.lock 一致，避免版本漂移
```

### 6. 卸載依賴
```bash
# 卸載生產依賴
pipenv uninstall requests

# 卸載開發依賴
pipenv uninstall --dev pytest

# 卸載所有依賴
pipenv uninstall --all
```

### 7. 查看虛擬環境信息
```bash
pipenv --venv  # 查看虛擬環境的路徑
pipenv --where  # 查看當前專案目錄（Pipfile 所在位置）
pipenv graph  # 查看依賴樹（顯示包之間的依賴關係）
```

### 8. 安全檢查
```bash
pipenv check  # 檢查依賴的安全漏洞和版本兼容性
```

### 9. 刪除虛擬環境
```bash
pipenv --rm  # 刪除當前專案的虛擬環境
```

## 四、Pipfile 與 Pipfile.lock 解析
### 1. Pipfile 範例
```toml
[[source]]
url = "https://pypi.org/simple"  # 包源（可替換為國內源，如豆瓣）
verify_ssl = true
name = "pypi"

[packages]
requests = "==2.31.0"  # 生產依賴
flask = "*"  # 接受任意版本

[dev-packages]
pytest = ">=7.0.0"  # 開發依賴

[requires]
python_version = "3.10"  # 指定 Python 版本
```

### 2. Pipfile.lock 範例（核心是 hash 和版本鎖定）
包含所有包的 SHA256 哈希、精確版本、依賴樹，確保安裝的包未被篡改且版本一致。

## 五、常見配置與優化
### 1. 更換國內源（解決安裝慢）
修改 `Pipfile` 的 `[[source]]` 部分：
```toml
[[source]]
url = "https://pypi.douban.com/simple"  # 豆瓣源
verify_ssl = true
name = "douban"
```

### 2. 指定 Python 版本
```bash
# 初始化時指定 Python 版本
pipenv install --python 3.10

# 或修改 Pipfile 的 [requires] 部分
[requires]
python_version = "3.10"
```

### 3. 忽略 Pipfile.lock（不推薦）
若需靈活更新依賴，可跳過鎖定文件：
```bash
pipenv install --skip-lock
```

## 六、與傳統方式的對比
| 特性         | pip + virtualenv            | Pipenv                |
| ------------ | --------------------------- | --------------------- |
| 虛擬環境管理 | 手動創建/激活               | 自動管理              |
| 依賴分類     | 單一 requirements.txt       | 分離生產/開發依賴     |
| 版本鎖定     | 需手動維護 requirements.txt | 自動生成 Pipfile.lock |
| 依賴衝突檢測 | 無                          | 內建解析              |
| 安全檢查     | 需額外工具                  | 內建 pipenv check     |

## 七、常見問題
1. **Pipenv 命令找不到**：確保 Python 的 user 目錄加入環境變量（如 Linux 的 `~/.local/bin`）。
2. **虛擬環境創建失敗**：檢查指定的 Python 版本是否安裝，或使用 `pipenv install --python $(which python3)`。
3. **安裝包速度慢**：更換國內源（如豆瓣、阿里）。
4. **Pipfile.lock 衝突**：協作時先執行 `pipenv lock` 再提交，或協商更新依賴。

## 八、總結
Pipenv 解決了傳統 Python 依賴管理的痛點，適用於中小型專案、團隊協作或需要可重現環境的場景。它的核心價值是「自動化虛擬環境 + 精確版本鎖定」，讓依賴管理更規範，減少「在我電腦上能跑」的問題。

若專案規模極大（如數百個依賴），可考慮 `poetry`（另一款現代包管理工具），但 Pipenv 對新手更友好、學習成本更低。