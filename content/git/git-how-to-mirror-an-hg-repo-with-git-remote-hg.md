Title: 使用 git-remote-hg 镜像一个 Hg 仓库
Slug: git-how-to-mirror-an-Mercurial-hg-repo-with-git-remote-hg
Date: 2015-08-16

前段时间在 github 上建了一个 PyPy 的 [镜像仓库](https://github.com/mozillazg/pypy) （官方 PyPy 仓库是个存放在  bitbucket 上的 Mercurial(Hg) 仓库）。本文将记录我使用 git-remote-hg 镜像 Hg 仓库的步骤以及后续的同步更新操作。

1. 安装 git 2.x （如果系统上的 git 是 2.x 版本可以跳过本步骤）。由于我用的是 CentOS 6 系统，通过 yum 安装的 git 版本是 1.x ，所以需要编译安装 2.x 版本的 git。详细步骤在这里: [在 CentOS 6 上编译安装 git 2.x](http://mozillazg.com/2015/08/install-git-2-on-centos-6.html)

2. 安装 Python 包：mercurial

        cd /tmp
        wget https://bootstrap.pypa.io/get-pip.py
        python get-pip.py
        pip install mercurial

3. 安装 [git-remote-hg](https://github.com/fingolfin/git-remote-hg)

        mkdir /usr/local/bin
        curl -o /usr/local/bin/git-remote-hg https://raw.githubusercontent.com/fingolfin/git-remote-hg/master/git-remote-hg
        chmod +x /usr/local/bin/git-remote-hg

4. 配置 `PATH` 环境变量:

        echo 'export PATH=/usr/local/bin:$PATH' >> ~/.bashrc
        source ~/.bashrc

5. 配置 git:

        git config --global core.notesRef refs/notes/hg

6. clone Hg 仓库，假设仓库存放在 /www

        cd /www
        git clone "hg::https://mozillazg@bitbucket.org/pypy/pypy"

7. 配置 github 仓库地址

        cd /www/pypy/
        git remote add mine git@github.com:mozillazg/pypy.git

8. 同步分支信息

    1. 下载 [fetch_remote_branch_name_to_local.py](https://github.com/mozillazg/snippets/blob/master/git/track_all_remote_branches.py)
    2. 执行脚本
    
            cd /www/pypy/
            python fetch_remote_branch_name_to_local.py

9. push 到 github 仓库

        git push mine --all
        git push mine --tags

10. 定时同步代码

    1. 下载同步分支信息的脚本： [fetch_remote_branch_name_to_local.py](https://github.com/mozillazg/snippets/blob/master/git/track_all_remote_branches.py)
    2. 将下面的代码同步脚本保存为 update.sh:

            #!/usr/bin/env bash

            source ~/.bashrc

            echo "step 0"
            cd /www/pypy/

            echo "step 1"
            for remote in `git branch|grep -v '\* master'`; do git branch -d $remote ;  done

            echo "step 2"
            git fetch origin

            echo "step 3"
            python track_all_remote_branches.py

            echo "step 4"
            git pull

            echo "step 5"
            git push mine --all -f

            echo "step 6"
            git push mine --tags -f
   
    3. 配置 crontab 定时任务，每天 3 点同步一次代码

            $ crontab -e

            0 3 * * * cd /www/pypy && /usr/bin/env bash /www/pypy/update.sh > /var/log/mirror_pypy.log 2>&1


想抢先查看实际效果？欢迎围观我创建的 [PyPy 镜像仓库](https://github.com/mozillazg/pypy)


## 参考资料

* <https://github.com/fingolfin/git-remote-hg>
* <https://bitbucket.org/pypy/pypy>
* <https://pip.pypa.io/en/latest/installing.html>
* <https://github.com/mozillazg/pypy>