# Ghi chép về rsyslog


# 1. Tổng quan

Rsyslog (rocket-fast system for log processing) là một phần mềm mã nguồn mở nhận tất cả các log message từ kernel và các application, hơn nữa nó có thể chuyển tiếp các log message đến các địa chỉ khác trên mạng. 

Rsyslog có hiệu suất cao, bảo mật tốt và thiết kế theo module. 

Các khả năng của rsyslog: 

- Chấp nhận lượng lớn các input từ nhiều nguồn 
- Chuyển tiếp chúng
- Output có thể đẩy đến nhiều nơi

Các log message được phân loại ở `/var/log`

# 2. Cấu hình

Mặc định rsyslog đã được cài trên Ubuntu. 

Rsyslogd được cấu hình trong file `rsyslog.conf` ở `/etc`

Có thể tạo file cấu hình dựa trên [tool online](http://www.rsyslog.com/rsyslog-configuration-builder/)

# 3. Cài đặt `rsyslog` trên Remote Server

Trên Remote Server, phải cấu hình để máy lắng nghe trên cổng sẽ gửi log, mặc định là port 514. Uncomment 2 dòng cấu hình sử dụng UDP hoặc TCP:

	# provides UDP syslog reception
	#$ModLoad imudp
	#$UDPServerRun 514

	# provides TCP syslog reception
	#$ModLoad imtcp
	#$InputTCPServerRun 514

Sau đó restart lại rsyslog:

	$ service rsyslog restart 
	
Trên Local Rsyslog
	
Cần cấu hình cho server ship log tới Remote Server. Sửa file `50-default.conf` hoặc tạo một file mới với đuôi .conf trong `/etc/rsyslog.d` và đặt dòng này :
	
	*.*   @remote.server:514
		


	
	
