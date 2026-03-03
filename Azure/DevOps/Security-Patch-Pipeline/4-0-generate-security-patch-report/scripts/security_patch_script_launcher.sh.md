# scripts/security_patch/security_patch_script_launcher.sh



## Code

```bash
LAUNCHER_SCRIPT_PATH=$(dirname "$0")
PYTHON_SCRIPT_NAME="$1"
shift

export PYTHONUNBUFFERED=TRUE
export PYTHONPATH="${LAUNCHER_SCRIPT_PATH}/../code_integration:${LAUNCHER_SCRIPT_PATH}/../common:${LAUNCHER_SCRIPT_PATH}"
python3 -m pipenv run python3 ${LAUNCHER_SCRIPT_PATH}/${PYTHON_SCRIPT_NAME} "$@"
```



這個 bash 腳本 [security_patch_script_launcher.sh](vscode-file://vscode-app/usr/share/code/resources/app/out/vs/code/electron-browser/workbench/workbench.html) 是一個 **Python 腳本啟動器**，用於在 pipenv 虛擬環境中執行 Python 腳本。以下是逐行說明：

## 腳本功能說明

**第 1 行**: `LAUNCHER_SCRIPT_PATH=$(dirname "$0")`

- 取得當前腳本所在的目錄路徑
- `$0` 是腳本本身的路徑，`dirname` 提取其目錄部分

**第 2 行**: `PYTHON_SCRIPT_NAME="$1"`

- 將第一個命令列參數（要執行的 Python 腳本名稱）存入變數

**第 3 行**: `shift`

- **移除第一個參數**，使剩餘參數可以傳遞給 Python 腳本

  - **執行前的參數狀態**：

    - `$1` = `apply_security_patch.py`
    - `$2` = `--branch`
    - `$3` = `main`
    - `$4` = `--spl`
    - `$5` = `2024-01`
    - `$@` = `apply_security_patch.py --branch main --spl 2024-01`

    **執行 `shift` 後的參數狀態**：

    - `$1` = `--branch` (原本的 `$2`)
    - `$2` = `main` (原本的 `$3`)
    - `$3` = `--spl` (原本的 `$4`)
    - `$4` = `2024-01` (原本的 `$5`)
    - `$@` = `--branch main --spl 2024-01`

**第 5 行**: `export PYTHONUNBUFFERED=TRUE`

- 設定 Python 以無緩衝模式運行
- 確保輸出即時顯示，不會被緩衝

**第 6 行**: `export PYTHONPATH="..."`

- 設定 Python 模組搜尋路徑，包含：
  - `../code_integration` - 上層的 code_integration 目錄
  - `../common` - 上層的 common 目錄
  - 當前腳本目錄本身
- 讓 Python 能找到這些目錄中的模組

**第 7 行**: `python3 -m pipenv run python3 ${LAUNCHER_SCRIPT_PATH}/${PYTHON_SCRIPT_NAME} "$@"`

- 使用 pipenv 虛擬環境執行指定的 Python 腳本
- `"$@"` 傳遞所有剩餘的命令列參數給 Python 腳本

## 使用範例

這會在 pipenv 環境中執行 `apply_security_patch.py`，並傳遞 `--arg1 value1 --arg2 value2` 參數。