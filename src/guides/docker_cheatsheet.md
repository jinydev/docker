---
layout: docs
title: "도커 & 컴포즈 핵심 명령어 치트시트"
permalink: /guides/docker_cheatsheet/
---

# 🐳 도커 & 컴포즈 핵심 명령어 치트시트 (Cheat Sheet)

## 1. 컨테이너 기본 생애주기
```bash
docker container run -d --name <이름> -p <호스트포트>:<컨테이너포트> <이미지>
docker container ls -a
docker container stop <이름/ID>
docker container start <이름/ID>
docker container rm -f <이름/ID>
docker container logs -f <이름/ID>
docker container exec -it <이름/ID> sh
```

## 2. 이미지 빌드 & 레지스트리
```bash
docker image build -t <태그명>:<버전> .
docker image ls
docker image prune -a
docker image push <레지스트리>/<이미지명>:<태그>
```

## 3. 볼륨 & 네트워크
```bash
docker volume create <볼륨명>
docker volume ls
docker volume inspect <볼륨명>
docker network create --driver bridge <네트워크명>
docker network ls
```

## 4. 도커 컴포즈 (Compose V2)
```bash
docker compose up -d
docker compose ps
docker compose logs -f
docker compose down -v
```
