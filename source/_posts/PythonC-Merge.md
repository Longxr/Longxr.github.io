---
title: Python合并C++文件
categories:
  - Python
tags:
  - Python
  - C++
  - Merge
date: 2018-09-27 17:38:26
---

<Excerpt in index | 摘要> 
Python合并C++文件<!-- more -->
<The rest of contents | 余下全文>

### 问题
 项目中接口类文件数量太多，老大让把同一个模块的接口文件合并到一个文件中，手动太特么累了，上Python。

### 代码 

```
# -*- coding: utf-8 -*-

import os,sys
info = os.getcwd()+'/MerageFiles'
fout_h = open('merge.h', 'w') # 合并.h到该文件
fout_cpp = open('merge.cpp', 'w') # 合并.cpp到该文件

def writeintofile_h(filepath, shotname):
	fin = open(filepath)
	# strinfo = fin.read()
	lines = fin.readlines()
	fout_h.write('\n/*********************************************************************************')
	fout_h.write('\n' + shotname)
	fout_h.write('\n*')
	fout_h.write('\n*********************************************************************************/\n')
	# fout_h.write(strinfo)
	for l in lines:
		if 'MANAGER_H' in l:
			continue
		elif '#include' in l:
			continue
		else:
			fout_h.write(l)
	fin.close()

def writeintofile_cpp(filepath, shotname):
	fin = open(filepath)
	# strinfo = fin.read()
	lines = fin.readlines()
	fout_cpp.write('\n/*********************************************************************************')
	fout_cpp.write('\n' + shotname)
	fout_cpp.write('\n*')
	fout_cpp.write('\n*********************************************************************************/\n')
	# fout_cpp.write(strinfo)
	for l in lines:
		if 'MANAGER_H' in l:
			continue
		elif '#include' in l:
			continue
		else:
			fout_cpp.write(l)
	fin.close()

for root, dirs, files in os.walk(info):
	if len(dirs)==0:
		for file in files:
			filepath = os.path.join(root,file)
			(shotname,extension) = os.path.splitext(file);
			if extension == '.h':
				writeintofile_h(filepath, shotname)
			elif extension == '.cpp':
				writeintofile_cpp(filepath, shotname)

fout_h.close()
fout_cpp.close()
```

### 使用
1. 新建个py文件拷贝代码，在py同目录下新建个`MerageFiles`文件夹用来放要合并的h、cpp文件。
2. 运行py文件。
