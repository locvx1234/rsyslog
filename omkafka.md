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
	
	
## Một số action parameter 

	
broker [brokers]

    Type: Array

    Default: “localhost:9092”

    Xác định các broker sử dụng 

topic [topicName]

    Xác định tên topic để produce

key [key]

    Default: none

    Kafka key được sử dụng cho tất cả các message

dynatopic [boolean]

    Default: off

    Nếu thiết lập, các  topic parameter sẽ trở thành một template cho việc produce các message. Cache sẽ được xóa trên HUP.

dynatopic.cachesize [positiveInteger]

    Default: 50

    Nếu thiết lập, xác định số topic sẽ được giữ trong bộ nhớ cache dynatopic.

partitions.auto [boolean]

    Default: off

    librdkafka cung cấp một function partition tự động nó sẽ tách đều các produced message ra tất cả các partition được cấu hình cho các topic.

    Để sử dụng, set partitions.auto=”on”. 

partitions.number [positiveInteger]

    Default: none

    Nếu thiết lập, xác định số partition tồn tại và thực hiện load-balancing. 
	
partitions.usedFixed [positiveInteger]

    Default: none

    Nếu thiết lập, xác định partition data được produce. Tất cả data sẽ được đi đến partition này.

errorFile [filename]

    Default: none

    Nếu thiết lập, các message không thể gửi được và gây ra một thông báo lỗi, ghi vào file được chỉ định, file này là định dạng JSON.

confParam [parameter]

    Type: Array

    Default: none

    Cho phép xác định các Kafka option. Thay vì cung cấp vô số các thiết lập để phù hợp với các thông số Kafka, thiết lập này như một phương tiện để thiết lập bất kỳ Kafka parameter nào, điều này hữu ích khi Kafka ra những bản release mới.
	Lưu ý, rsyslog sử dụng librdkafka cho kết nối Kafka.
	
topicConfParam [parameter]

    Type: Array

    Default: none

    Về cơ bản tương tự như confParam nhưng đối với các topic Kafka

template [templateName]

    Default: template set via “template” module parameter

closeTimeout [positiveInteger]

    Default: 2000

    Thời gian đợi (milliseconds) cho việc dựng submit các message (cung cấp bởi librdkafka) trước khi ngắt kết nối.

	
*Tham khảo*

https://dzone.com/articles/recipe-rsyslog-kafka-logstash-1	

http://www.rsyslog.com/doc/master/configuration/modules/omkafka.html?highlight=omkafka
