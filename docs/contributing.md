

项目地址：[https://github.com/111rran/ranwiki](https://github.com/111rran/ranwiki)

可在 Wiki 页面右上角打开本项目。

本项目仅授予11ran相关同学参考。

## 参与方式

### 在 GitHub 中直接编辑文档

在对应的文件页面，点击右上角的小铅笔按钮，即可直接在浏览器中修改，修改完成后填写 commit 信息提交。

### 将项目 `git clone` 到本地

推荐使用 SSH 协议：

```bash
git clone git@github.com:Xiamen-University-Malaysia/xcpc-wiki.git
```

注意：需要先在 GitHub 上设置 SSH Key，具体步骤可以参考 GitHub 官方文档：[Connecting to GitHub with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)。

或者使用 HTTPS 协议：

```bash
git clone https://github.com/Xiamen-University-Malaysia/xcpc-wiki.git
```

克隆到本地后，使用文本编辑器进行编辑。修改完成后，参考下方的 [`命名规范`](#_3) 和 [`提交 commit 规范`](#commit) 进行操作。如果有新增专栏或、增文章或更改导航、更改左侧目录，需要修改 [`mkdocs.yml`](../mkdocs.yml) 文件。

推送到远程仓库前，建议在本地预览修改效果。预览前确保安装 MkDocs 及其相关依赖：

```bash
pip install -r requirements.txt
```

安装完成后，在项目的根目录下运行以下命令启动本地服务器：

```bash
mkdocs serve
```

访问终端中输出的本地地址，访问即可预览修改后的网站，通常为：[http://127.0.0.1:8000/](http://127.0.0.1:8000/)

最后通过 `git push` 推送到 GitHub 远程仓库：

```bash
git add .
git commit -m "Add P1001 solution"
git push origin main
```

或使用 GUI 工具进行提交和推送。


## 提交 commit 规范

提交 commit 前确保代码或文档已经过测试和验证，能够正常运行。

关于 commit message 的规范，请使用以下格式：

- Add: 添加新内容

  Add P1001 solution

- Update: 更新现有内容

  Update index.md file

- Fix: 修复错误

  Fix P1001 solution

- Refactor: 重构代码或文档

  Refactor code structure

- Format: 调整代码或文档格式

  Format code style

- Remove: 删除内容

  Remove unused code

- MkDocs: 与 MkDocs 配置相关的更改

  MkDocs update navigation structure

## 格式指南

### 使用简单块（Admonition）

!!! note "标题"
    块内容

``` markdown
!!! note "标题"
    块内容
```

### 使用可折叠块（Details）

??? question "点击展开"
    折叠的内容

``` markdown
??? question "点击展开"
    折叠的内容
```

支持的块类型：`note`，`abstract`，`info`，`tip`，`warning`，`failure`，`success`，`question`，`danger`，`bug`，`example`，`quote`。

### 代码行高亮

如果需要突出代码中的重要行，可以使用 `hl_lines` 参数指定要高亮的行号：

高亮单行：

```markdown
```cpp hl_lines="2"
```

```cpp hl_lines="2"
#include <bits/stdc++.h>
using namespace std;  // 这一行会被高亮

int main() {
    return 0;
}
```

高亮多行：

```markdown
```cpp hl_lines="5 6 7"
```

```cpp hl_lines="5 6 7"
#include <bits/stdc++.h>
using namespace std;

int main() {          // 这三行会被高亮
    int n = 10;
    cout << n << endl;
    return 0;
}
```

高亮行范围：

```markdown
```cpp hl_lines="5-7"
```

```cpp hl_lines="5-7"
#include <bits/stdc++.h>
using namespace std;

int main() {          // 这一段会被高亮
    int n = 10;
    cout << n << endl;
    return 0;
}
```

### 显示行号

使用 `linenums="1"` 参数可以在代码块前显示行号，便于引用和说明：

带行号的代码：

```markdown
```cpp linenums="1"
```

```cpp linenums="1"
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n = 10;
    cout << n << endl;
    return 0;
}
```

结合行高亮：


```markdown
```cpp linenums="1" hl_lines="4 8"
```

```cpp linenums="1" hl_lines="4 8"
#include <bits/stdc++.h>
using namespace std;

int main() {          // 这两行会被高亮
    int n = 10;
    cout << n << endl;
    return 0;
}
```

