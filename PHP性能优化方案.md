高并发，分布式，高性能设计（性能问题？）
> 思路 + 想法 操刀方法

* 性能测试工具
	1，ab压力测试（Apache benchmark）
		> /ab -n1000 -c100 http://www.baidu.com 
				请求总数量   并发数
		> 指标：每秒响应数 + 每个请求响应时间 
		
	2，XHProf（源自Facebook的调试工具）
		* php -ri xhprof - 查看是否安装完成

	3，vld - opcache代码分析

* 优化的方向
	> time php a.php //time直接用linux的计算脚本执行时间
	1，语言级代码结构和代码相关优化
		* 多用PHP内置函数，少自定义
		* 内置函数性能差异，关注
		* 少用魔法函数
		* 少用@屏蔽错误，用php -dvid扩展查看PHP执行的opcode
		* 合理使用内存，及时释放
		* 正则使用小心
		* 不适合密集型业务
		
		
	2，周边环境优化，比如mysql，nginx，第三方服务
		* 服务器系统
		* 磁盘
		* 数据库
		* IO效率最低
			1，读写内存  -》  读取数据库 -》 读取磁盘 -》 读取网络
			2，socket网络接口，在linux里面也是基于文件
			3，规避读取大文件，网络差的
		* 优化网络请求
			1，设置超时
				* 连接超时 200ms - 正常国内机房，这个值已经很大了
				* 读超时 800ms
				* 写超时 500ms
			2，将串行请求并行
				* curl可以设置 - curl_multi_*()
				* swoole
		
	3，终极方案，PHP底层C代码优化
		* opcache 的缓存技术 apc
		* c扩展替代PHP实现
		* runtime优化（关注）
		
* PHP7性能优化

	JIT = JUST IN TIME, 二进制机器码

	//大幅度优化
	1，zval使用栈内存
	2，zend_string存储hash值，array查找不在需要重复计算hash

	//额外
	3，hashtable桶内直接存数据，减少内存申请次数，提高cache命中率和内存访问速度
	4，zend_parse_parameters改为宏命令，性能提高5%
	5，新增4种opcode，call_user-function,is_array/is_int/is_string,definded，速度更快
	6，基础类型优化，int，string，float，直接改为值拷贝，以前是引用计数
	7，排序算法优化，php官方重写排序算法

		