---
layout: post
title: "Cách sửa lỗi đồng hồ garmin 935 kết nối GPS chậm"
date: 2019-05-24 12:00:00.000000000 +07:00
author: "Dương Dâu"
type: post
published: true
status: publish
categories: 
- Running
tags:
- running
- garmin
- gps
- forunner 935
excerpt: "Cách sửa lỗi đồng hồ garmin 935 kết nối GPS chậm."

---
## Hiện tượng
Chiếc đồng hồ garmin của bạn bỗng nhiên kết nối GPS rất chậm từ 1-2 phút thậm chí 5-6 phút thay vì 5-10s mặc dù đang đứng ở chỗ thoáng ??? 

Hiện tượng trên hay sảy ra nhất là sau 1 thời gian không thực hiện activity, không đồng bộ garmin với điện thoại, máy tính .

![garminslowgps]( {{site.url}}/assets/img/2019/05/24/run.png)

## Nguyên nhân và cách kiểm tra

Trên đồng hồ garmin có 1 file là EPO (Extended Prediction Orbit) là file dự đoán quỹ đạo của vệ tinh. File này giúp garmin khi bật GPS sẽ biết được vệ tinh ở đâu và có thể kết nối ngay đến các vệ tinh gần nhất mà không cần dò tìm, do đó garmin chỉ mất 5-10s để kết nối GPS.

Tuy nhiên file này chỉ có tác dụng trong 1 khoảng thời gian ngắn từ vài ngày - 1 tuần, sau khoảng thời gian này, file EPO sẽ hết hạn (expired), khi đó garmin sẽ phải dò lại GPS mất rất nhiều thời gian từ 1-2 phút thậm chí lên đến 5-6 phút.

Cách kiểm tra: Trên đồng hồ vào mục Settings - About - cuộn xuống dưới sẽ có thông tin trạng thái EPO, nếu trạng thái là expired tức là file EPO đã hết hạn.

![garminepoexpired]( {{site.url}}/assets/img/2019/05/24/expired.PNG)

## Cách khắc phục

File EPO này sẽ được update khi kết nối đồng hồ với phần mềm Garmin Express trên máy tính hoặc kết nối đồng hồ qua wifi.

Lưu ý: Việc kết nối đồng hồ garmin với điện thoại dùng phần mềm Garmin Connect sẽ KHÔNG update được file EPO đã expired, do đó chỉ thực hiện được bằng 1 trong 2 cách trên.

Sau khi update kiểm tra lại thì trạng thái EPO phải là current:

![garminepocurrent]( {{site.url}}/assets/img/2019/05/24/current.png)

Sau khi trạng thái EPO là current rồi thì việc bắt GPS sẽ nhanh trở lại.
