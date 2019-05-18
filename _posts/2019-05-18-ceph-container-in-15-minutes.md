---
layout: post
title: "Xây dựng hệ thống Ceph storage cluster trong container trong 15 phút"
date: 2019-05-18 12:00:00.000000000 +07:00
author: "Dương Dâu"
type: post
published: true
status: publish
categories: 
- Tutorials
tags:
- ceph
- container
- docker
excerpt: "Xây dựng hệ thống Ceph storage cluster sử dụng Docker."

---

Sau đây tôi trình bày các xây dựng 1 hệ thống Ceph storage cluster trong container đơn giản.

## Ceph storage cluster là gì

[Ceph storage cluster](https://ceph.com/) là giải pháp mã nguồn mở cung cấp dịch vụ lưu trữ phân tán, sử dụng trên nhiều loại phần cứng (server, ổ cứng) phổ thông. Có thể hiểu Ceph storage cluster được sinh ra để thay thế các giải pháp SAN đắt tiền.
Ceph bao gồm 1 số thành phần chính như sau:
* Monitor: Là thành phần lưu trữ trạng thái Cluster, bao gồm lưu trữ monitor map, osd map, placement group map, ...
* OSD: Là thành phần lưu trữ dữ liệu chính, làm việc trực tiếp với ổ cứng, chịu trách nhiệm đảm bảo đồng bộ dữ liệu
* Mgr: Là thành giám sát việc sử dụng tài nguyên của cluster, cung cấp các thông tin như tổng dung lượng hiện có, dung lượng đã sử dụng, cung cấp các metric để giám sát performance


## Chuẩn bị

Tôi dựng hệ thống này trên 1 server chạy OS [Ubuntu 16.04](http://releases.ubuntu.com/16.04/). Các thành phần của Ceph được chạy dưới dạng [Docker Container](https://www.docker.com/) nhằm dễ dàng trong việc triển khai, quản lý cũng như nâng cấp.
Server chạy Ceph cluster cần ít nhất 1 ổ cứng còn trống, lưu ý không dùng chung với ổ cứng cài OS. Trong các ví dụ dưới đây tôi sử dụng ổ cứng /dev/sdb

Chạy script dưới đây để cài đặt docker container:

```bash
apt-get update
apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
apt-get update
apt-get install -y docker-ce 
```

Tôi sử dụng Ceph phiên bản mimic, theo kinh nghiệm của tôi nên sử dụng phiên bản stable mới nhất - 1, để đảm bảo độ ổn định của hệ thống, ví dụ phiên bản stable mới nhất là nautilus thì tôi sẽ sử dụng phiên bản mimic.

```bash
docker pull ceph/daemon:latest-mimic
```

## Cài đặt

Đầu tiên cần chạy dịch vụ Ceph Monitor

```bash
docker run -d --net=host --name=ceph-mon \
-v /etc/ceph:/etc/ceph \
-v /var/lib/ceph:/var/lib/ceph \
-e MON_IP=10.10.0.100 \
-e CEPH_PUBLIC_NETWORK=10.10.0.0/24 \
ceph/daemon:latest-mimic mon
```
Trong đó thay giá trị MON_IP bằng địa chỉ IP hiện tại của node chạy Ceph Monitor, CEPH_PUBLIC_NETWORK là địa chỉ network của node chạy Ceph Monitor

Tiếp theo chạy dịch vụ Ceph Mgr

```bash
docker run -d --net=host --name=ceph-mgr \
-v /etc/ceph:/etc/ceph \
-v /var/lib/ceph:/var/lib/ceph \
ceph/daemon:latest-mimic mgr
```

Chuẩn bị phân vùng lại ổ cứng chạy Ceph OSD

```bash
docker run -d --privileged=true --name=ceph-zap-sdb \
-v /dev/:/dev/ \
-e OSD_DEVICE=/dev/sdb \
ceph/daemon:latest-mimic zap_device
```

Cuối cùng là chạy dịch vụ OSD trên ổ cứng vừa phân vùng

```bash
docker run -d --net=host --name=ceph-osd-sdb \
--privileged=true \
--pid=host \
-v /etc/ceph:/etc/ceph \
-v /var/lib/ceph:/var/lib/ceph \
-v /dev/:/dev/ \
-e OSD_DEVICE=/dev/sdb \
-e OSD_TYPE=disk \
ceph/daemon:latest-mimic osd
```

Kiểm tra lại bằng lệnh ceph -s trong Ceph monitor nếu thấy health là HEALTH_OK và có 1 osd up & in là được
```bash
root@ubuntu:~# docker exec -ti ceph-mon ceph -s
  cluster:
    id:     34776b39-a4bb-4316-94e5-225884208763
    health: HEALTH_OK
 
  services:
    mon: 1 daemons, quorum ubuntu
    mgr: ubuntu(active)
    osd: 1 osds: 1 up, 1 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   2.0 GiB used, 277 GiB / 279 GiB avail
    pgs:     
```

## Cấu hình ceph dashboard

Việc theo dõi hiện trạng của Ceph storage cluster hoàn toàn có thể thực hiện bằng dòng lệnh "ceph-s" hoặc "ceph -w" như ở trên. Tuy nhiên để có thể theo dõi trực quan hơn thì dịch vụ Ceph mgr có cung cấp giao diện dashboard để có thể theo dõi hiện trạng cluster từ xa

Vào chế độ dòng lệnh của Ceph monitor và thực hiện các lệnh sau
```bash
root@ubuntu:~# docker exec -ti ceph-mon bash
[root@ubuntu /]# ceph mgr module enable dashboard
[root@ubuntu /]# ceph dashboard create-self-signed-cert
[root@ubuntu /]# ceph dashboard set-login-credentials admin password
```
Trong đó thay admin/password bằng username/password truy cập dashboard. 
Truy cập địa chỉ <https://10.10.0.100:8080> đăng nhập bằng tài khoản vừa tạo, trong dashboard này tôi có thể hoàn toàn theo dõi hiện trạng cluster

![Dashboard]( {{site.url}}/assets/img/2019/05/18/ceph_dashboard_02.PNG)

## Xóa cluster
Trong trường hợp cần xóa cluster thì thực hiện các lệnh sau
```bash
docker rm -f -v ceph-mon ceph-mgr ceph-osd-sdb ceph-zap-sdb
rm -rf /etc/ceph
rm -rf /var/lib/ceph
```

