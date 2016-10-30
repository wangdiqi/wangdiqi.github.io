---
layout: post
title: "cmake学习"
subtitle: "CMakeLists.txt 的编写"
date: 2016-10-30 16:35:00
author: "seventhking"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    -- cmake
---

> "学会编写CMakeLists.txt"

## CMakeLists.txt 的变量

PROJECT(projectname [CXX] [C] [Java])



example1:
PROJECT(HELLO)
SET(SRC_LIST main.c)
MESSAGE(STATUS "this is binary dir" ${HELLO_BINARY_DIR})
MESSAGE(STATUS "this is source dir" ${HELLO_SOURCE_DIR})
ADD_EXECUTABLE(hello ${SRC_LIST})


