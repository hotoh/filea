@echo off
:begin
set /p var_branch=请输入同步的分支编号：

git remote remove main
git remote prune origin

set remote_branch=remotes/main/%var_branch%
set local_branch=remotes/origin/%var_branch%
set remote_branch_exists=0
set local_branch_exists=0

for /f  %%i in ('git branch -a') do ( 
    setlocal enabledelayedexpansion
    set curr_branch=%%i
	if !curr_branch!==%local_branch% (
        set "local_branch_exists=1"
    )
)

if !local_branch_exists! equ 1  (
        echo 分支已经存在，直接执行 "git checkout 分支名称" 即可!!
		goto :end
    )


for /f "tokens=6 delims=./" %%j in ('git remote -v') do (
    setlocal enabledelayedexpansion
    set project=%%j

    set git_addr=ssh://git@isource-nj.huawei.com:2222/BES-CSMF/!project!.git
    echo !git_addr!

    git remote add main !git_addr! 
    git fetch main -v  
    

    ::检查输入的分支编号，是否正确
    set target_branch=%var_branch%
    for /f  %%k in ('git branch -a') do ( 
        setlocal enabledelayedexpansion
        set curr_branch2=%%k

        if !curr_branch2!==%remote_branch% (
            set "remote_branch_exists=1"
        )

    )
	
	goto :checkout
)

:checkout
if !remote_branch_exists! equ 0  (
        echo 分支名称不正确，请检查分支名称!!
		git remote remove main
		goto :end
    )

    git branch %var_branch% main/%var_branch%
    git checkout %var_branch%
    git pull main %var_branch%
    git push origin %var_branch%
    git remote remove main

	goto :end
	
:end
	pause
	@popd
	@endlocal
	@echo on
	
