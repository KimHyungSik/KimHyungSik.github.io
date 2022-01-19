---
layout: post
title:  GithubActions 사용해 보기
categories: [Github, CI, Github Actions]
tags: [ Github Actions, Github]
comments: true 
---

GithubActions란 
-----------

github에서 컴퓨터를 대여 wrokflow를 자동화 하는 도구이다
대표적인 사례로
  - 테스트 코드
  - 서버, 앱 등을 배포 
  - 코드 스타일 검사
  - CI/CD를 구현 할 수 있다
```
name: CI
```
Github Actions 네이밍

```
on:

  pull_request_target:
    branches:
      - master
      - develop
```         
어떤 상황에이 actions를 실행 시킬지
  - push, pull_request등 상황을 정의
  - branches로 어떤 branch에서 위의 상황이 생길때 정의([master, develop]로 array로 작성할 수도 있음)

```
jobs:
```
Workflow는 다양한 job로 구성 하여 실행 한다

```
runs-on: ubuntu-latest
```
job을 실행 시킬 환경을 선택 한다

```
 - name: Unit test
```
job안에서 실행 시키는 단위들에도 네이밍을 할 수 있다 

```
run: ./gradlew test
```
run은 실행 시킬 명령어를 입력한다

```
uses: actions/checkout@v1
```
uses는 내가 설정한 run이 아닌 github 에서 제공해 주는 marketplace에서 다른 사용자들이 만들어 놓은 workflow들을 가져와 실행 시킨다

```
name: CI

on:

  pull_request_target:
    branches:
      - master
      - develop

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1

      - name: Setup JDK 11
        uses: actions/setup-java@v2
        with:
          java-version: '11'
          distribution: 'adopt'

      - name: set up JDK 1.8
        uses: actions/setup-java@v1
        with:
          java-version: 1.8

      - name: Grant Permissions to gradlew
        run: chmod +x gradlew

      - name: Unit test
        run: ./gradlew test
      
      - name: Ktlint
        run: ./gradlew ktlint
```
내가 간단하게 만들어 놓은 workflow이다
android에서 모든 테스트 코드를 실행 시키고
Ktlint의 코드스타일을 검사 하는 wrokflow
