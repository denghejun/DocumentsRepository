## Git (bash)
######当本地误删除了某文件时
* 单个文件
     git checkout App1/a.php
* 当前目录所有文件
     git checkout .

###### Undo操作
* 还未执行(git add .)的新增文件操作Undo：git clean -fdx (注意：修改文件和删除文件将不会被undo)
* 执行了(git add .或git commit)的所有操作（增/删/改）的Undo：
     * undo到本地前一次commit的结果(只能一次有效)：git reset --hard
     * undo到本地指定一次commit的结果：git reset --hard HEAD~index（注：index是commit数组的索引,所以是从0开始的，比如HEAD~0指的是最后一次的commit；HEAD~1是倒数第二次的commit；）
     * undo到Server端最后一次commit的结果：git reset --hard @{u}（注：此命令执行时，有可能Server端已经被其他人push过了，但它不会主动undo到最新的版本，所以，理论上讲，该命令会undo到本地最后一次成功push的版本，这个原理其实跟大多数源码管理工具如：svn、VSTS等是一样的，undo操作并不等价于undo + git newest。）
* 所以，一般情况下若为了确保完全undo所有操作，需要连续执行以下2条cmd：git clean -fdx | git reset --hard @{u}.就可以恢复到git服务器版本。

###### check out
* 切换到一个已经存在的分支: git checkout your-branch-name
* 创建一个不存在的分支并切换到该分支: git chekout -b your-branch-name
* 分支重命名：
```
git branch -m old_branch new_branch         # Rename branch locally    
git push origin :old_branch                 # Delete the old branch    
git push --set-upstream origin new_branch   # Push the new branch, set local branch to track the new remote
```


###### push & pull
* 当执行push推送时，若发现server端的文件与本地不是最新的时，将会push失败，此时执行：git pull拉取最新的server端版本后方可成功。

* 如果git commit了多次，但是都还没有push，这时输入：git push即可全部push，而使用git push origin master则报错。
* 有一次我git的密码修改了，我是执行这个命令：git config --global credential.helper，之后，就会在push时，重新输入密码了

###### auto complete
* 在`terminal`中当输入不完整的名词时，按下`Tab`键，将会自动补全；最长用的场景就是操作那些较长的分支名称。

## NuGet
###### proxy setting
* 当你所处的开发环境需要用代理上网时，需要设置代理，Nuget才能正常Restore一些Package，打开 `%appdata%\NuGet`(一般类似C:\Users\ld71\AppData\Roaming\NuGet，%appdata%是环境变量，指向Users\{userName}\AppData\Roaming),添加以下设置(具体值按需设置)：
```
<configuration>
     <config>
        <add key="http_proxy" value="http://my.proxy.address:port" />
        <add key="http_proxy.user" value="mydomain\myUserName" />
        <add key="http_proxy.password" value="base64encodedHopefullyEncryptedPassword" />
     </config>
</configuration>
```
###### pack package
打包并上传一个`.csproj`的Nuget包分四步（在`.csproj`文件目录下进行）

* `nuget spec`: 生成`xxx.nuspec`文件(这一步一般意义只需执行一次，之后就修改些内容就可以了，比如ReleaseNotes)

* 修改`xxx.nuspec`文件中的必要信息，如Author、ReleaseNotes等，版本号在Pack会自动替换

* `nuget pack xxx.csproj  -IncludeReferencedProjects`： 生成`xxx.1.0.0.x.nupkg`文件；`-IncludeReferencedProjects`参数指示包含该项目所有 的依赖项目（比如xxx.csproj依赖了yyy.dll，package在上传时会包含它），但一般若我们的项目对外的依赖都通过Nueget来管理，所以这个参数基本不会使        用，Nuget会在Install时自动解析依赖。除非我们有本地的dll，而又不能或不想使用Nuget来管理的话就适用该参数。

* `nuget push xxx.1.0.0.x.nupkg -s "http://10.16.76.251:9999/NugetServer/" "Newegg.Shipping.NugetServer"`: push Package; 

###### pack files
一般来讲，若上传的Package依赖使用方的一些配置文件（如app.config）则需要将文件名后追加`.transform`（如app.config.transform），然后在`.nuspec`文件中添加对应`files`节点(注意：是直接在`<package>`节点下)：
```
<?xml version="1.0"?>
<package xmlns="http://schemas.microsoft.com/packaging/2012/06/nuspec.xsd">
  <files>
     <!--src: 文件在当前项目的相对位置；-->
     <!--target: 文件在打包好的Package内的相对位置(Package Install时会将'content'文件夹下的所有目录和文件直接Install到目标项目)-->
     <file src="Configuration\Database.config.transform" target="content\Configuration\" />
	<file src="Configuration\App.config.transform" target="content\" />
  </files>
</package>
```

## VS Code
###### switch project
* 为了让程序编写时智能感知能够正常（比如按下`Ctrl+.`后能够提示`using`），或调试时监控变量更详细，最好将Current Project切换到你工作的项目下，有2种方式

     * `Ctrl+Shift+P`，然后输入select project，回车后即可看到root目录下所有的projects（project.json）
     * 在编辑器的右下角，有一个`火焰`一样的图标，点击它即可切换

###### debug
* .NET Core项目，当需要调式程序时，设置`launch.json`文件，如下(program属性指向了需要调试的程序)：
```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": ".NET Core Launch (console)",
            "type": "coreclr",
            "request": "launch",
            "preLaunchTask": "build",
            "program": "${workspaceRoot}/implement/bin/Debug/netcoreapp1.0/win10-x64/implement.dll",
            "args": [],
            "cwd": "${workspaceRoot}",
            "externalConsole": false,
            "stopAtEntry": false,
            "internalConsoleOptions": "openOnSessionStart"
        },
        {
            "name": ".NET Core Attach",
            "type": "coreclr",
            "request": "attach",
            "processId": "${command.pickProcess}"
        }
    ]
}
```

* 调试程序启动前，一般需要执行build task，若需要调试的project(project.json)不在`${workspaceRoot}`目录下，则需按以下形式在`task.json`文件中指定：
```
{
    "version": "0.1.0",
    "command": "dotnet",
    "isShellCommand": true,
    "args": [],
    "tasks": [
        {
            "taskName": "build",
            "args": [
                "${workspaceRoot}/implement"
            ],
            "isBuildCommand": true,
            "problemMatcher": "$msCompile"
        }
    ]
}
```
