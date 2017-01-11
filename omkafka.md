# Gửi log sang Kafka

Rsyslog là một phần mềm thiết kế dạng module, nó có thể chuyển tiếp các message tới các máy khác và các ứng dụng khác.

Bài viết này sẽ cấu hình cho rsyslog chuyển tới Kafka bằng việc sử dụng module `omkafka`

Trong file config `/etc/rsyslog.conf`, thêm module `omkafka` và `imfile`:

	module(load="imuxsock")  # will listen to your local syslog
	module(load="imfile")    # if you want to tail files
	module(load="omkafka")   # lets you send to Kafka

Với OS cũ hơn (Ubunut 14.04) module sẽ được định dạng 

	$ModLoad omkafka
	$ModLoad imfile 

Nếu muốn tail file, bạn phải thêm định nghĩa cho mỗi nhóm file:

	input(type="imfile"
		File="/opt/logs/example*.log"
		Tag="examplelogs"
	)


Nếu bạn cần format sang dạng JSON, thêm template : \

	template(name="json_lines" type="list" option.json="on") {
	  constant(value="{")
	  constant(value="\"timestamp\":\"")
	  property(name="timereported" dateFormat="rfc3339")
	  constant(value="\",\"message\":\"")
	  property(name="msg")
	  constant(value="\",\"host\":\"")
	  property(name="hostname")
	  constant(value="\",\"severity\":\"")
	  property(name="syslogseverity-text")
	  constant(value="\",\"facility\":\"")
	  property(name="syslogfacility-text")
	  constant(value="\",\"syslog-tag\":\"")
	  property(name="syslogtag")
	  constant(value="\"}")
	}
	
Mặc định rsyslog có queue 10K message, và có thread đơn, làm việc với lô (batch) 16 message. Bạn có thể thay đổi số thread, số batch, kích thước queue như sau : 


	main_queue(
	  queue.workerthreads="1"      # threads to work on the queue
	  queue.dequeueBatchSize="100" # max number of messages to process at once
	  queue.size="10000"           # max queue size
	)
	
Cuối cùng, để publish tới Kafka, phải xác định broker, topic muốn gửi tới : 

	action(
	  broker=["localhost:9092"]
	  type="omkafka"
	  topic="rsyslogkafka"
	  template="json"
	)

Restart rsyslog để chấp nhận thay đổi:

	$ service rsyslog restart 
	
Test: 

Trên Kafka server, sử dụng console-consumer để thấy log gửi về 
	
	$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic rsyslogkafka --from-beginning

Nếu không nhận được log, đọc log của rsyslog tại `/etc/log/rsyslog.log` và comment tại bài viết này để trao đổi thêm.
	
	
*Tham khảo*

https://dzone.com/articles/recipe-rsyslog-kafka-logstash-1	


	
