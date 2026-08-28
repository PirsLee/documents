# clone指定版本文件


如果只想拉取该版本、不要完整历史（CI/只读场景）

git clone --branch v1.2.0 --depth 1 https://github.com/user/repo.git

# 取消暂存区的所有文件

git reset HEAD

# 取消暂存区的单个文件

git reset HEAD filename.txt