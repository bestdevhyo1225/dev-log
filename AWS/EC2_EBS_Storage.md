# EC2 에서 사용중인 EBS 용량 늘리기

AWS 콘솔에서 EBS 용량을 늘린 후의 처리 방법을 정리한다.

-  `lsblk` 명령어로 인스턴스에 연결된 블록 디바이스 확인하기

```shell
lsblk
```
 
- 볼륨은 커졌어도 파티션 크기를 늘려야한다.

```shell
sudo growpart <볼륨> <파티션번호>
# ex) sudo growpart /dev/xvda 1
```

- `ext4` 파일 시스템이라면, 아래의 명령어로 크기를 늘려야한다.

```shell
sudo resize2fs <파티션>
# ex) sudo resize2fs /dev/xvda1
```

## 참고 문서

- [ec2 용량 full일 때 대처법](https://velog.io/@hyeonseop/ec2-%EC%9A%A9%EB%9F%89-full%EC%9D%BC-%EB%95%8C-%EB%8C%80%EC%B2%98%EB%B2%95)
