---
title: jenkins集成部署环境构建
date: 2023-01-26 12:52:41
tags:
categories:
---

更新机制是指项目如何进行更新，主要有两种方式：一种是自动推送，另外一种是手动拉取。前者用于开发环境、后者可以用于所有环境

# 1. 手动拉取

拉取更新流程：

<img src="https://panyuro.oss-cn-beijing.aliyuncs.com/20230126125309.png" style="zoom:40%;" />

1. sudo -i 输入密码，进入root目录
2. 上述流程由 deploy.sh 脚本实现：

```sh
#!/bin/bash -e
cd "`dirname $0`"
. ./pom.sh

#1. download war, ready env
echo "deploy time: $work_time"
mkdir -p war/
war=war/$pom_a-$pom_v.war
download_path="$nexus_redirect?r=$pom_r&g=$pom_g&a=$pom_a&v=$pom_v&e=war"
wget  $download_path -O $war

deploy_war
```

pom.sh 脚本内容：

```xml
#!/bin/bash -e
. ../bin/env-set.sh

pom_g=org.example
pom_a=test-jenkins
pom_v=1.0-SNAPSHOT
pom_r=snapshots


deploy_war() {
        target_d=war/${pom_a}-${pom_v}-$work_time
        target_dir=`pwd`/$target_d
        if [ ! -f "$war" ]; then
                echo "war not exist: $war"
                exit 1
        fi
        unzip -q $war -d $target_dir
        rm -rf appwar
        ln -sf $target_d appwar

        ./tomcat.sh stop

        target_ln=`pwd`/appwar
        echo '<?xml version="1.0" encoding="UTF-8" ?>
<Context docBase="'$target_ln'" allowLinking="true">
</Context>' > conf/Catalina/localhost/ROOT.xml
        ./tomcat.sh start
}
```

tomcat.sh 内容：

```sh
#!/bin/bash
if [ "`whoami`" != "root" ];then
		echo "Error: You must be apps to run this command."
		exit 1
fi

cd "`dirname $0`"
. ../bin/env-set.sh
. ./pom.sh

export CATALINA_BASE="`pwd`"
tomcat "$1" "${pom_a}-${pom_v}"
```

 env-set.sh内容：

```sh
#!/bin/bash -e
#当前时间
export now_time=$(date +%Y-%m-%d_%H-%M-%S)
#整个期间的开始时间
[ -z "$work_time" ] && export work_time=$(date +%Y-%m-%d_%H-%M-%S)

if [ -z "$g_env_set" ]; then
	unset JRE_HOME JAVA_HOME CLASSPATH
	export nexus_redirect="http://localhost:8081/nexus/service/local/artifact/maven/redirect"
	export data_home="/var/root/svr"
	export server_home="$data_home/services"
	export JAVA_HOME="$data_home/jdk"
	export CATALINA_HOME="$data_home/apache-tomcat"
	export CATALINA_BASE="$CATALINA_HOME"
	export PATH=$JAVA_HOME/bin:$PATH
	export g_env_set=true
fi

#能够输入2个grep的字符串用来查找
proc_find() {
        if [ "$#" = 2 ]; then
                ps -eo pid,cmd|grep -v 'cmd\|grep'|grep "$1"|grep "$2"|sed 's/^ *\(.*\) *$/\1/'
        else
                ps -eo pid,cmd|grep -v 'cmd\|grep'|grep "$1"|sed 's/^ *\(.*\) *$/\1/'
        fi
}
#能够输入2个grep的字符串用来查找
proc_pid() {
        proc_find "$@" | cut -d ' ' -f1
}
#$1: 要重试的次数，超过此次数后使用kill -9
#$2: 能够输入2个grep的字符串用来查找
proc_kill() {
        grep_args="${@:2}"
        for ((i=1;i<=$1;++i))
        do
                pids=(`proc_pid "$grep_args"`)
                [ ${#pids[@]} = 0 ] && return 0

                echo "n=$i kill ${pids[@]}"
                kill ${pids[@]}
                sleep 1s
        done
        pids=(`proc_pid "$grep_args"`)
        [ ${#pids[@]} = 0 ] || kill -9 ${pids[@]}
}

tomcat_start0() {

	[ "$JAVA_OPTS" = "" ] && export JAVA_OPTS="-server -XX:MaxPermSize=128m -Xms512m -Xmx512m"
	[ "`echo "$JAVA_OPTS"|grep '\-Djava.security.egd='`" = "" ] && export JAVA_OPTS="$JAVA_OPTS -Djava.security.egd=file:/dev/./urandom"
#防止X11导致awt在X11关闭后异常
	export JAVA_OPTS="-Djava.awt.headless=true $JAVA_OPTS -$grep_server"
	echo "START $grep_server: JAVA_OPTS=$JAVA_OPTS"
	"$CATALINA_HOME"/bin/startup.sh
}
tomcat_stop0() {
	proc_kill 3 "$grep_server"
}
tomcat_status0() {
	tmp=`proc_find "$grep_server"`
	if [ -z "$tmp" ]; then
		echo "$grep_server stopped."
	else
		echo "$grep_server running: $tmp"
	fi
}

tomcat() {
	if [ "$1" = "" ]; then
		printf 'Usage: %s {start|stop|restart|status} ????\n' "$prog"
		exit 1
	fi
	if [ "$2" = "" ]; then
		echo 'java -Dmykey=${mykey} miss: mykey'
		exit 1
	fi
	if [ "$CATALINA_BASE" = "" ]; then
		echo 'miss: $CATALINA_BASE='
		exit 1
	fi

	cd "$CATALINA_BASE"
	export grep_server="Dmykey=${2}x"
	case "$1" in
	start)
		tmp=`proc_find "$grep_server"`
		if [ -z "$tmp" ]; then
			tomcat_start0
		else
			echo "$grep_server already running: $tmp"
		fi
		sleep 1s
		#tomcat_status0
		;;
	stop)
		tomcat_stop0
		tomcat_status0
		;;
	status)
		tomcat_status0
		;;
	restart)
		tomcat_stop0
		tomcat_start0
		sleep 1s
		tomcat_status0
		;;
	*)
		printf 'Usage: %s {start|stop|restart|status} ????\n' "$prog"
		exit 1
		;;
	esac
}
```

**代码地址**：[pyr9/jenkins-script (github.com)](https://github.com/pyr9/jenkins-script)
