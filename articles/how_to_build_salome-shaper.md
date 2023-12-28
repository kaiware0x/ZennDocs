---
title: "CAEツール「SALOME」のSHAPERモジュールをビルドする"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cpp", "cae", "visualstudio", "cmake", "salome"]
published: true
---


## はじめに

オープンソースのCAE(Computer Aided Enginnering)ツールとして、
**SALOME**がある。
https://www.salome-platform.org

今回はこのSALOMEの**SHAPERモジュール**という1つの機能をソースコードからビルドする。
SHAPERはSALOME内でパラメトリック3D-CAD機能を提供している。

ビルド方法について公式情報がほとんど無く、かなり苦戦したのでメモとしてこの記事を残す。


## 環境

- OS: Windows11
- Compiler: MSVC
  - Build Tool for Visual Studio 2017 が必要。インストール方法は後述
- Gitがインストール済みで`git.exe`へPATHが通っている
- SALOMEバージョン: 9.11.0


## SALOMEのダウンロード

このページからダウンロードする。

https://www.salome-platform.org/?page_id=2430

以下のように選択し、Submitボタンを押してダウンロードする。
**WINDOWS 10 (.EXE) ではなく、WINDOWS 10 (.ZIP) を選択する。**

![](/images/how_to_build_salome-shaper/2023-12-28-12-03-33.png)

ダウンロードしたらできるだけ**階層が浅いフォルダ**へ展開する。
今回は `C:/Work/s`フォルダ(今後**ROOTDIR**と呼ぶ)にZIPを展開した。


現状のフォルダ構成
```
C:/Work/s (←ROOTDIR)
├── SALOME-9.11.0
└── SALOME-9.11.0-6df795577bb4bb61a32d6ba7f5dc2b7f.zip
```


## ビルドツールのClone [^1]

次の内容を`SALOMEConfigTool.bat`としてROOTDIRに保存する。

:::details SALOMEConfigTool.bat
```bat
@echo off

REM 
REM SALOME archive to developer move 
REM

REM tag à utiliser
set TAG_SAT=master
set TAG_SAT_SALOME=master
set SAT_APPLICATION_NAME=SALOME-master
set CURRENT_DIRECTORY=%cd%

REM SAT and SAT_SALOME GIT repositories
set SAT_GIT_REPOSITORY=https://git.salome-platform.org/gitpub/tools/sat.git
set SAT_SALOME_GIT_REPOSITORY=https://git.salome-platform.org/gitpub/tools/sat_salome.git

:PARSE_OPTIONS
  if not "%1"=="" (
    if "%1"=="--tag_sat" (
        set TAG_SAT=%2
    )
        shift
    if "%1"=="--tag_sat_salome" (
        set TAG_SAT_SALOME=%2
        shift
    )
    if "%1"=="--application" (
        set SAT_APPLICATION_NAME=%2
	shift
    )
    if "%1"=="--help" (
        goto :SYNOPSIS
    )
    shift
    goto :PARSE_OPTIONS
  )

echo.
echo INFO: TAG_SAT..........= %TAG_SAT%
echo INFO: TAG_SAT_SALOME...= %TAG_SAT_SALOME%
echo INFO: APPLICATION......= %SAT_APPLICATION_NAME%

REM
REM Check that git is installed
REM 
echo.
echo INFO: checking if git.exe program is in your PATH environment variable
SET GIT_ROOT_DIR=
FOR /F "tokens=*" %%g IN ('where git') do (SET GIT_ROOT_DIR=%%g) 
echo.
echo INFO: found git = %GIT_ROOT_DIR%

IF NOT DEFINED GIT_ROOT_DIR (
    echo.
    echo FATAL: Could not find git.exe command. Install Git and add it to PATH environment variable!
    goto :END_OF_SCRIPT
)

if not EXIST %SAT_APPLICATION_NAME% (
  echo.
  echo FATAL: Please download %SAT_APPLICATION_NAME% and extract it in directory: %CURRENT_DIRECTORY%...
  goto :END_OF_SCRIPT
)

REM
REM clone SAT
REM
cd %CURRENT_DIRECTORY%
if EXIST SAT (
  echo.
  echo INFO: removing existing SAT directory...
  RMDIR /Q /S SAT
)

cd %CURRENT_DIRECTORY%
git clone %SAT_GIT_REPOSITORY% SAT
cd SAT
git checkout %TAG_SAT%

REM
REM clone SAT_SALOME
REM
cd %CURRENT_DIRECTORY%
if exist SAT_SALOME (
  echo.
  echo INFO: removing existing SAT_SALOME directory...
  rmdir /Q /S SAT_SALOME
)
git clone %SAT_SALOME_GIT_REPOSITORY% SAT_SALOME
cd SAT_SALOME
git checkout %TAG_SAT_SALOME%

echo.
echo INFO : renaming salome.pyconf
cd %CURRENT_DIRECTORY%
copy /Y SAT_SALOME\salome-W10.pyconf SAT_SALOME\salome.pyconf

echo.
echo INFO: renaming application %SAT_APPLICATION_NAME%-windows.pyconf yo %SAT_APPLICATION_NAME%.pyconfcd applications
copy /Y SAT_SALOME\applications\%SAT_APPLICATION_NAME%-windows.pyconf SAT_SALOME\applications\%SAT_APPLICATION_NAME%.pyconf

REM Ajout des projets dans SAT
cd %CURRENT_DIRECTORY%\SAT
echo.
echo INFO: add SAT_SALOME project to SAT

%CURRENT_DIRECTORY%\%SAT_APPLICATION_NAME%\W64\Python\python.exe .\sat init --add_project  %CURRENT_DIRECTORY%\SAT_SALOME\salome.pyconf
REM Ajout de l'url et du tag
echo.
echo INFO: add repository to VCS list: %SAT_GIT_REPOSITORY%
%CURRENT_DIRECTORY%\%SAT_APPLICATION_NAME%\W64\Python\python.exe .\sat init --VCS %SAT_GIT_REPOSITORY%
echo.
echo INFO: define SAT Tag in use...
FOR /F %%i IN ('git describe --tags') DO set SAT_GIT_TAG=%%i
%CURRENT_DIRECTORY%\%SAT_APPLICATION_NAME%\W64\Python\python.exe .\sat init --tag %SAT_GIT_TAG%

echo.
echo INFO: SAT tag: %SAT_GIT_TAG%

REM remove products
echo.
echo INFO: setting default git server to https://git.salome-platform.org/gitpub/modules
cd %CURRENT_DIRECTORY%\SAT_SALOME\applications
powershell -Command "(Get-Content -ReadCount 0 %SAT_APPLICATION_NAME%.pyconf) -replace 'repo_dev : \"yes\"', 'repo_dev : \"no\"' | Set-Content %SAT_APPLICATION_NAME%.pyconf"

echo.
echo INFO: removing RESTRICTED product...
cd %CURRENT_DIRECTORY%\SAT_SALOME\applications
powershell -Command "(Get-Content -ReadCount 0 %SAT_APPLICATION_NAME%.pyconf) -replace \"'RESTRICTED'\", \"#'RESTRICTED'\" | Set-Content %SAT_APPLICATION_NAME%.pyconf"

echo.
echo INFO: removing CEA TEST BASE product...
cd %CURRENT_DIRECTORY%\SAT_SALOME\applications
powershell -Command "(Get-Content -ReadCount 0 %SAT_APPLICATION_NAME%.pyconf) -replace \"'CEATESTBASE'\", \"#'CEATESTBASE'\" | Set-Content %SAT_APPLICATION_NAME%.pyconf"

echo INFO: patching %CURRENT_DIRECTORY%\%SAT_APPLICATION_NAME%. Be patient...
set diskTg=E:\S
if "%SAT_APPLICATION_NAME%" == "SALOME-9.6.0" (
  set diskTg=D:\S
)

REM patching [i]
set oldStr="%diskTg%\SAT_SALOME"
set newStr="%CURRENT_DIRECTORY%\SAT_SALOME"
set searchStr=%oldStr:\=\\%
set replaceStr=%newStr:\=\\%
echo INFO: replacing %searchStr% with %replaceStr% . Be patient...
cd %CURRENT_DIRECTORY%
powershell -Command "Get-ChildItem %CURRENT_DIRECTORY%\%SAT_APPLICATION_NAME% -Recurse -Include \"*.pyconf\" | ForEach-Object { (Get-Content $_.FullName) | ForEach-Object { $_ -replace [regex]::Escape(\"%searchStr%\"), \"%replaceStr%\" } | Set-Content $_.FullName }"

set oldStr="%diskTg%\%SAT_APPLICATION_NAME%"
set newStr="%CURRENT_DIRECTORY%\%SAT_APPLICATION_NAME%"

REM patching [ii]
set searchStr=%oldStr%
set replaceStr=%newStr%
echo INFO: replacing %searchStr% with %replaceStr% . Be patient...
cd %CURRENT_DIRECTORY%
powershell -Command "Get-ChildItem %CURRENT_DIRECTORY%\%SAT_APPLICATION_NAME% -Recurse -Include \"*.bat\",\"*.py\",\"*.pyconf\" | ForEach-Object { (Get-Content $_.FullName) | ForEach-Object { $_ -replace [regex]::Escape(\"%searchStr%\"), \"%replaceStr%\" } | Set-Content $_.FullName }"

REM patching [iii]
set searchStr=%oldStr:\=\\%
set replaceStr=%newStr:\=\\%
echo INFO: replacing %searchStr% with %replaceStr% . Be patient...
cd %CURRENT_DIRECTORY%
powershell -Command "Get-ChildItem %CURRENT_DIRECTORY%\%SAT_APPLICATION_NAME% -Recurse -Include \"*.pyconf\" | ForEach-Object { (Get-Content $_.FullName) | ForEach-Object { $_ -replace [regex]::Escape(\"%searchStr%\"), \"%replaceStr%\" } | Set-Content $_.FullName }"

REM patching [iv]
set searchStr=%oldStr:\=/%
set replaceStr=%newStr:\=/%
echo INFO: replacing %searchStr% with %replaceStr% . Be patient...
cd %CURRENT_DIRECTORY%
powershell -Command "Get-ChildItem %CURRENT_DIRECTORY%\%SAT_APPLICATION_NAME% -Recurse -Include \"*.bat\",\"*.py\",\"*.pyconf\",\"*.cmake\" | ForEach-Object { (Get-Content $_.FullName) | ForEach-Object { $_ -replace [regex]::Escape(\"%searchStr%\"), \"%replaceStr%\" } | Set-Content $_.FullName }"

cd %CURRENT_DIRECTORY%

echo.
echo INFO: %SAT_APPLICATION_NAME% is now setup in development mode.
echo INFO: setup Visual Studio environment.
echo INFO call "C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Auxiliary\Build\vcvarsall.bat" x64

goto :END_OF_SCRIPT


:SYNOPSIS
  echo SYNOPSIS: call %~n0%~x0
  echo OPTIONS:
  echo   --tag_sat        : SALOME Tool tag or branch name, default value set to : %TAG_SAT%
  echo   --tag_sat_salome : SAT_SALOME tag or branch name, default value set to  : %TAG_SAT_SALOME%
  echo   --application    : SAT_SALOME application name, default value set to    : %SAT_APPLICATION_NAME%
  echo   --help           : this help
  echo.
  goto :END_OF_SCRIPT
	
:END_OF_SCRIPT
  cd %CURRENT_DIRECTORY%
```
:::

ROOTDIRをPowerShellで開き、以下のコマンドを実行する。
実行には`git.exe`へのPATHが通っている必要がある。

```bat
./SALOMEConfigTool.bat --application SALOME-9.11.0
```

上記バッチファイルを実行すると以下のリポジトリがCloneされる。
また各フォルダ内のファイルパスを自分用のファイルパスに修正してくれる。

- ビルドツール`SAT` (SAlome Toolsの略)
  - Pythonスクリプトの集合体
  - SALOMEのビルドもSATを経由して行う
  - 詳しくは`{ROOTDIR}/SAT/pdf/salomeTools.pdf`を参照
- ビルド設定ファイル群`SAT_SALOME`
  - SATを用いてビルドするための各種設定が書かれたファイル`.pyconf`が大量に含まれている
    - ビルド内容を変更したい場合は対象モジュールの`.pyconf`の書き換えが必要


現状のフォルダ構成
```
C:/Work/s (←ROOTDIR)
├── SALOMEConfigTool.bat
├── SAT
├── SAT_SALOME
├── SALOME-9.11.0
└── SALOME-9.11.0-6df795577bb4bb61a32d6ba7f5dc2b7f.zip
```


## パッチを当てる [^1]

次の内容を`sat_system.patch`ファイルとしてROOTDIRに保存する。

:::details sat_system.patch
```patch
diff --git a/src/system.py b/src/system.py
index 17a7c06..da8d9e1 100644
--- a/src/system.py
+++ b/src/system.py
@@ -152,7 +152,7 @@ exit $res
     # NOTICE: this command only works with recent version of git
     #         because --work-tree does not work with an absolute path
     if src.architecture.is_windows():
-      cmd = "rm -rf %(where)s && git clone %(git_options)s %(remote)s %(where)s && git --git-dir=%(where_git)s --work-tree=%(where)s checkout %(tag)s"
+      cmd = "rmdir /S /Q %(where)s && git clone %(git_options)s %(remote)s %(where)s && git --git-dir=%(where_git)s --work-tree=%(where)s checkout %(tag)s"
     else:
 # for sat compile --update : changes the date of directory, only for branches, not tag
       cmd = r"""
@@ -244,13 +244,13 @@ rm -rf $tmpDir
     cmd = r"""
 
 set tmpDir=%(tmpWhere)s && \
-rm -rf $tmpDir
-git clone %(git_options)s %(remote)s $tmpDir && \
-cd $tmpDir && \
+rmdir /S /Q %tmpDir%
+git clone %(git_options)s %(remote)s %tmpDir% && \
+cd %tmpDir% && \
 git checkout %(tag)s && \
 mv %(sub_dir)s %(where)s && \
-git log -1 > %(where)s/README_git_log.txt && \
-rm -rf $tmpDir
+git log -1 > %(where)s\\README_git_log.txt && \
+rmdir /S /Q %tmpDir%
 """ % aDict
 
   DBG.write("cmd", cmd)
```
:::

`{ROOTDIR}/SAT`フォルダを**Git Bash**で開き、以下のコマンドを実行する。
Git Bashを開くにはGitをインストールし、右クリックメニューへの追加をしておく必要がある。

```bash
patch -p1 < ../sat_system.patch
```

またバッチファイルかpatchの不具合(?)で次のPythonスクリプトが崩れてしまっているので修正する。

`{ROOTDIR}/SALOME-9.11.0/W64/Python/lib/sre_compile.py`
50行目あたりでカッコ閉じがコメントに入ってしまっている。

```diff py:sre_compile.py
-    (0xfb05, 0xfb06), # �E�E��E)
+    (0xfb05, 0xfb06), # �E�E��E
+    )
```


## Build Tools for Visual Studio 2017 のインストール

SALOMEのビルドにはVisual Studio 2017時代のコンパイラが必要。
現状Community版はダウンロードできないので、ビルドツール単体をインストールする。
インストールにはMicrosoftアカウントが必要。(無料)

https://visualstudio.microsoft.com/ja/vs/older-downloads/

![](/images/how_to_build_salome-shaper/2023-12-28-13-13-49.png)


## ビルド実行

VS2017をインストールすると環境変数がセットされたコマンドプロンプトが追加される。
`VS 2017 用 x64 Native Tools コマンド プロンプト` を管理者として実行する。(通常実行でも良いかも)

![](/images/how_to_build_salome-shaper/2023-12-28-13-15-59.png)

ROOTDIRへ移動し、次の`sat prepare`コマンドを実行する。

```bash
SALOME-9.11.0\W64\Python\python.exe SAT\sat -t -v 6 prepare SALOME-9.11.0 -p SHAPER,CONFIGURATION
```

次の`sat config --list`コマンドを実行する。
実行結果の中に `SALOME-9.11.0` が表示されていることを確認する。

```bash
SALOME-9.11.0\W64\Python\python.exe SAT\sat config --list
```

次の`sat compile`コマンドでビルドを実行する。

```bash
SALOME-9.11.0\W64\Python\python.exe SAT\sat -t -v 6 compile SALOME-9.11.0 -p SHAPER,CONFIGURATION --clean_all
```

すべてOKで終了したことを確認する。


## アプリケーションの実行

`{ROOTDIR}/SALOME-9.11.0/BUILD/SHAPER`フォルダにVisual Studioのソリューションファイルができているので開く。

Visual Studio上部のビルド構成を**Release**にする。

![](/images/how_to_build_salome-shaper/2023-12-28-13-28-21.png)

`shapergui_app`というプロジェクトがあるので、**右クリック→スタートアップ プロジェクトに設定**を選択する。

![](/images/how_to_build_salome-shaper/2023-12-28-13-25-08.png)

`shapergui_app`プロジェクトについて **右クリック→プロパティ** を選択する。
**構成プロパティ→デバッグ→環境**からQtへのPATHを通す。

```
PATH=C:\Work\s\SALOME-9.11.0\W64\qt\bin;%PATH%
```

![](/images/how_to_build_salome-shaper/2023-12-28-13-27-54.png)


`shapergui_app.cpp`を次のように編集する。
変数`salome_root_dir`は個人の環境に合わせて修正する。

:::details shapergui_app.cpp
```cpp:shapergui_app.cpp
// Copyright (C) 2021-2023  CEA, EDF
//
// This library is free software; you can redistribute it and/or
// modify it under the terms of the GNU Lesser General Public
// License as published by the Free Software Foundation; either
// version 2.1 of the License, or (at your option) any later version.
//
// This library is distributed in the hope that it will be useful,
// but WITHOUT ANY WARRANTY; without even the implied warranty of
// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
// Lesser General Public License for more details.
//
// You should have received a copy of the GNU Lesser General Public
// License along with this library; if not, write to the Free Software
// Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307 USA
//
// See http://www.salome-platform.org/ or email : webmaster.salome@opencascade.com
//

#include <QProcessEnvironment>
#include <QDir>
#include <QProcess>
#include <QDebug>

#include <iostream>

int main(int argc, char* argv[])
{
	/// Set environment variables
	/// Reference: ~~~~/SALOME-9.11.0/salome (Python script file)

	auto pe = QProcessEnvironment::systemEnvironment();

	auto addToPATH = [&pe](const QString& value) {
		pe.insert("PATH", value + ";" + pe.value("PATH"));
		};
	auto addToPYTHONPATH = [&pe](const QString& value) {
		pe.insert("PYTHONPATH", value + ";" + pe.value("PYTHONPATH"));
		};


	/// [FIX ME]
	const QString salome_root_dir("C:/Work/s/SALOME-9.11.0/W64");

	/// [Python]
	const auto python_version = QString("3.6");
	const auto python_libdir = QString("lib/python%1/site-packages").arg(python_version);
	const auto python_home = QString("%1/Python").arg(salome_root_dir);
	pe.insert("PYTHON_VERSION", python_version);
	pe.insert("PYTHON_LIBDIR", python_libdir);
	pe.insert("PYTHONHOME", python_home);
	addToPATH(python_home);

	/// [KERNEL]
	{
		const auto kernel_root_dir = QString("%1/KERNEL").arg(salome_root_dir);
		pe.insert("KERNEL_ROOT_DIR", kernel_root_dir);
		addToPATH(QString("%1/bin/salome").arg(kernel_root_dir));
		addToPATH(QString("%1/lib/salome").arg(kernel_root_dir));
		addToPYTHONPATH(QString("%1/bin/salome").arg(kernel_root_dir));
		addToPYTHONPATH(QString("%1/lib/salome").arg(kernel_root_dir));
		addToPYTHONPATH(QString("%1/%2/salome").arg(kernel_root_dir).arg(python_libdir));
	}

	/// [GUI]
	{
		const auto gui_root_dir = QString("%1/GUI").arg(salome_root_dir);
		pe.insert("GUI_ROOT_DIR", gui_root_dir);
		addToPATH(QString("%1/bin/salome").arg(gui_root_dir));
		addToPATH(QString("%1/lib/salome").arg(gui_root_dir));
		addToPYTHONPATH(QString("%1/bin/salome").arg(gui_root_dir));
		addToPYTHONPATH(QString("%1/lib/salome").arg(gui_root_dir));
		addToPYTHONPATH(QString("%1/%2/salome").arg(gui_root_dir).arg(python_libdir));
	}

	/// [SHAPER]
	{
		const auto shaper_root_dir = QString("%1/SHAPER").arg(salome_root_dir);
		pe.insert("SHAPER_ROOT_DIR", shaper_root_dir);
		addToPATH(QString("%1/bin/salome").arg(shaper_root_dir));
		addToPATH(QString("%1/lib/salome").arg(shaper_root_dir));
		addToPYTHONPATH(QString("%1/bin/salome").arg(shaper_root_dir));
		addToPYTHONPATH(QString("%1/lib/salome").arg(shaper_root_dir));
		addToPYTHONPATH(QString("%1/%2/salome").arg(shaper_root_dir).arg(python_libdir));
	}

	/// [Others]
	{
		pe.insert("QT_QPA_PLATFORM_PLUGIN_PATH", QString("%1/qt/plugins/platforms").arg(salome_root_dir));
		pe.insert("SALOME_GUI_ROOT_DIR", QString("%1/GUI/bin/salome").arg(salome_root_dir));
		addToPATH(QString("%1/qt/bin").arg(salome_root_dir));
		addToPATH(QString("%1/CAS/win64/vc14/bin").arg(salome_root_dir));
		addToPATH(QString("%1/ParaView/bin").arg(salome_root_dir));
		addToPATH(QString("%1/EXT/bin").arg(salome_root_dir));
		addToPATH(QString("%1/OT/bin").arg(salome_root_dir));
		addToPATH(QString("%1/qwt/lib").arg(salome_root_dir));
		addToPATH(QString("%1/pthreads/lib").arg(salome_root_dir));
		addToPATH(QString("%1/omniORB/bin/x86_win32").arg(salome_root_dir));
		addToPATH(QString("%1/planegcs/lib").arg(salome_root_dir));
	}

	constexpr char GUI_ROOT_DIR[] = "GUI_ROOT_DIR";
	constexpr char SHAPER_ROOT_DIR[] = "SHAPER_ROOT_DIR";
	constexpr char LIGHTAPPCONFIG[] = "LightAppConfig";
	// QProcessEnvironment pe(QProcessEnvironment::systemEnvironment());
	if (!pe.contains(SHAPER_ROOT_DIR))
	{
		std::cerr << SHAPER_ROOT_DIR << " is not defined in your environment !" << std::endl;
		return 1;
	}
	if (!pe.contains(GUI_ROOT_DIR))
	{
		std::cerr << GUI_ROOT_DIR << " is not defined in your environment !" << std::endl;
		return 1;
	}
	// fill LightAppConfig env var
	QString gui_root_dir(QDir::fromNativeSeparators(pe.value(GUI_ROOT_DIR)));
	QString shaper_root_dir(QDir::fromNativeSeparators(pe.value(SHAPER_ROOT_DIR)));
	//QString lightappconfig_val(QString("%1:%2")
	QString lightappconfig_val(QString("%1;%2") // COLON to SEMICOLON
		.arg(QDir::toNativeSeparators(QString("%1/share/salome/resources/gui").arg(gui_root_dir)))
		.arg(QDir::toNativeSeparators(QString("%1/share/salome/resources/shaper")
			.arg(shaper_root_dir))));
	pe.insert(LIGHTAPPCONFIG, lightappconfig_val);
	//tells shutup to salome.salome_init invoked at shaper engine ignition
	pe.insert("SALOME_EMB_SERVANT", "1");

	const auto suitexe_path = QString("%1/bin/salome/suitexe.exe").arg(gui_root_dir);

	if (!QFile::exists(suitexe_path)) {
		std::cerr << suitexe_path.toStdString() << " does not exist!" << std::endl;
		return 1;
	}

	//
	QProcess proc;
	proc.setProcessEnvironment(pe);
	proc.setProgram(suitexe_path);
	proc.setArguments({ "--modules=SHAPER", "--no-splash" });
	proc.setProcessChannelMode(QProcess::ForwardedErrorChannel);
	proc.start();
	proc.waitForFinished(-1);
	return proc.exitCode();
}
```
:::

**Ctrl+F5**を押して実行すると、SHAPERモジュールが有効になった状態のSALOMEのGUIが表示される。

![](/images/how_to_build_salome-shaper/2023-12-28-14-19-44.png)


---

[^1]: https://discourse.salome-platform.org/t/how-to-build-shaper-in-window-11/505/1
