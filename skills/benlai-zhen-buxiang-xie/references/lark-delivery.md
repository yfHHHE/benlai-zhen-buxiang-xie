# 交付到飞书

只在用户要求放进飞书时使用。依赖 `lark-cli`。

## 新建

1. 以用户身份在个人空间新建，默认私有，不改分享设置：
   `lark-cli docs +create --as user --parent-position my_library --doc-format markdown --title "<标题>" --content @<去掉一级标题的 md>`
2. 确认链接分享是关闭的：`lark-cli api GET /open-apis/drive/v2/permissions/<token>/public --params '{"type":"docx"}' --as user`
3. 用户要发给自己时，用机器人私信把链接发给用户本人，不发给其他人或群。

**不要**替用户打开分享、移动到知识库、发到群里。这些都要用户自己决定和操作。

## 更新（防止覆盖用户的修改）

用户随时可能在飞书里直接改文档，或把文档移进知识库后继续改。每次更新前：

1. 读最新版本号：`lark-cli docs +fetch --as user --doc <token>`，取 `revision_id`
2. 和上次写入后的版本号比较。不一致时，用 `--revision-id` 分别拉取两个版本的 markdown 做 diff，看用户改了什么
3. 把用户的改动当作决定合进本地文件（删掉的名字、改过的标题、删掉的列、删掉的段落都要保留用户的版本）
4. 用 `--revision-id <刚读到的版本号>` 写入，这样期间再有人改会直接失败，不会覆盖：
   `lark-cli docs +update --as user --doc <token> --command overwrite --doc-format markdown --revision-id <N> --content @<file>`
5. 写完读回来核对：标题、表格数量、章节、敏感词（人名、具体主体名）

覆盖写入不会改标题；标题需要改时单独用 `--command str_replace`。

如果用户的改动被误覆盖了：用 `docs +history-list` 找到用户最后编辑的版本，用 `docs +fetch --revision-id` 取回，合并后再写入，并如实告诉用户。
