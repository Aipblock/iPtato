#!/usr/bin/env bash
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
#=================================================
#       System Required: CentOS/Debian/Ubuntu
#       Description: iptables 出封禁 入放行
#       Version: 1.0.20
#		Blog: 计划中
#=================================================

sh_ver="1.0.20"
Green_font_prefix="\033[32m"
Red_font_prefix="\033[31m"
Green_background_prefix="\033[42;37m"
Red_background_prefix="\033[41;37m"
Font_color_suffix="\033[0m"
Info="${green}[信息]${Font_color_suffix}"
Error="${Red_font_prefix}[错误]${Font_color_suffix}"

checkfile="/root/checkfile.txt"
smtp_port="25,26,465,587"
pop3_port="109,110,995"
imap_port="143,218,220,993"
other_port="24,50,57,105,106,158,209,1109,24554,60177,60179"
bt_key_word="torrent
.torrent
peer_id=
announce
info_hash
get_peers
find_node
BitTorrent
announce_peer
BitTorrent protocol
announce.php?passkey=
magnet:
xunlei
sandai
Thunder
XLLiveUD"

# check root
[[ $EUID -ne 0 ]] && echo -e "${Error} 必须使用root用户运行此脚本！\n" && exit 1

check_system() {
	if [[ -f /etc/redhat-release ]]; then
		release="centos"
	elif cat /etc/issue | grep -q -E -i "debian"; then
		release="debian"
	elif cat /etc/issue | grep -q -E -i "ubuntu"; then
		release="ubuntu"
	elif cat /etc/issue | grep -q -E -i "centos|red hat|redhat"; then
		release="centos"
	elif cat /proc/version | grep -q -E -i "debian"; then
		release="debian"
	elif cat /proc/version | grep -q -E -i "ubuntu"; then
		release="ubuntu"
	elif cat /proc/version | grep -q -E -i "centos|red hat|redhat"; then
		release="centos"
	fi
	bit=$(uname -m)
}
check_run() {
	runflag=0
	if [ ! -e "${checkfile}" ]; then
		touch $checkfile
		echo "首次运行判断文件生成"
		set_environment
		echo "初次运行脚本 环境部署完成"
	else
		runflag=1
		echo "文件存在 脚本不是初次运行"
	fi
}
shell_run_tips() {
	if [ ${runflag} -eq 0 ]; then
		echo
		echo "本脚本默认接管 控制出入网 权限"
		echo "入网端口仅放行了 SSH端口"
		echo
	fi
}

set_environment() {
	install_iptables
	install_tool
	long_save_rules_tool
	rebuild_iptables_rule
	able_ssh_port
}
install_iptables() {
	getiptables=$(iptables -V | awk 'NR==1{print  $1}')
	if [ "$release" == "debian" ] || [ "$release" == "ubuntu" ]; then
		if [ -z ${getiptables} ]; then
			apt-get install iptables -y
		fi
	elif [[ "$release" == "centos" ]] && [ -z ${getiptables} ]; then
		yum install iptables -y	
	fi
}
install_tool() {
	getnetstat=$(netstat --version | awk 'NR==1{print  $1}')
	if [ "$release" == "debian" ] || [ "$release" == "ubuntu" ]; then
		if [ -z ${getnetstat} ]; then
			apt install net-tools -y
		fi
	elif [[ "$release" == "centos" ]]; then
		if [ -z ${getnetstat} ]; then
			yum install net-tools -y
		fi
	fi
}
rebuild_iptables_rule() {
	iptables -P INPUT ACCEPT
	iptables -F
	iptables -A INPUT -m ttl --ttl-gt 80 -j ACCEPT
	iptables -A INPUT -p icmp -j ACCEPT
	iptables -A INPUT -i lo -j ACCEPT
	iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
	iptables -P INPUT DROP
}
long_save_rules_tool() {
	if [ "$release" == "debian" ] || [ "$release" == "ubuntu" ]; then
		echo "此类系统不需安装longruletool"
	elif [[ "$release" == "centos" ]]; then
		cipstatu=$(service iptables status | awk 'NR==1{print  $1}')
		firestatus="$(firewall-cmd --state)"
		if [ "${firestatus}" == "running" ]; then
			echo "停止firewall中"
			systemctl stop firewalld.service
			echo "禁止firewall开机启动"
			systemctl disable firewalld.service
			echo "成功关闭firewall"
		fi
		if [ -z ${cipstatu} ]; then
			yum install iptables-services -y
			systemctl enable iptables
		fi
	fi
}
able_ssh_port() {
	s="A"
	get_ssh_port
	set_in_ports
}
var_v4_v6_iptables() {
	v4iptables=$(iptables -V)
	v6iptables=$(ip6tables -V)
	if [[ ! -z ${v4iptables} ]]; then
		v4iptables="iptables"
		if [[ ! -z ${v6iptables} ]]; then
			v6iptables="ip6tables"
		fi
	else
		exit 1
	fi
}

# 查看出网模块
view_all_disable_out() {
	echo
	display_out_port
	display_out_keyworld
	echo
}

# 出网端口模块
disable_want_port_out() {
	s="A"
	input_disable_want_outport
	set_out_ports
	echo -e "${Info} 已封禁端口 [ ${PORT} ] !\n"
	disable_port_type_1="1"
	while true; do
		input_disable_want_outport
		set_out_ports
		echo -e "${Info} 已封禁端口 [ ${PORT} ] !\n"
	done
	display_out_port
}
input_disable_want_outport(){
	echo -e "请输入欲封禁的 出网端口（单端口/多端口/连续端口段）"
	if [[ ${disable_port_Type_1} != "1" ]]; then
	echo -e "${Green_font_prefix}========出网端口示例说明========${Font_color_suffix}
 单端口：25（单个端口）
 多端口：25,26,465,587（多个端口用英文逗号分割）
 连续端口段：25:587（25-587之间的所有端口）" && echo
	fi
	read -e -p "(回车默认取消):" PORT
	[[ -z "${PORT}" ]] && echo "已取消..." && display_out_port && exit 0
}
set_out_ports() {
	if [[ -n "$v4iptables" ]] && [[ -n "$v6iptables" ]]; then
		tcp_outport_rules $v4iptables $PORT $s
		udp_outport_rules $v4iptables $PORT $s
		tcp_outport_rules $v6iptables $PORT $s
		udp_outport_rules $v6iptables $PORT $s
	elif [[ -n "$v4iptables" ]]; then
		tcp_outport_rules $v4iptables $PORT $s
		udp_outport_rules $v4iptables $PORT $s
	fi
	save_iptables_v4_v6
}
tcp_outport_rules() {
	[[ "$1" = "$v4iptables" ]] && $1 -t filter -$3 OUTPUT -p tcp -m multiport --dports "$2" -m state --state NEW,ESTABLISHED -j REJECT --reject-with icmp-port-unreachable
	[[ "$1" = "$v6iptables" ]] && $1 -t filter -$3 OUTPUT -p tcp -m multiport --dports "$2" -m state --state NEW,ESTABLISHED -j REJECT --reject-with tcp-reset
}
udp_outport_rules() {
	$1 -t filter -$3 OUTPUT -p udp -m multiport --dports "$2" -j DROP
}

able_want_port_out() {
	s="D"
	input_able_want_outport
	set_out_ports
	echo -e "${Info} 已取消封禁端口 [ ${PORT} ] !\n"
	able_port_type_1="1"
	while true; do
		input_able_want_outport
		set_out_ports
		echo -e "${Info} 已取消封禁端口 [ ${PORT} ] !\n"
	done
	display_out_port
}
input_able_want_outport() {
	echo -e "请输入欲取消封禁的 出网端口（单端口/多端口/连续端口段）"
	if [[ ${able_port_Type_1} != "1" ]]; then
	echo -e "${Green_font_prefix}========出网端口示例说明========${Font_color_suffix}
 单端口：25（单个端口）
 多端口：25,26,465,587（多个端口用英文逗号分割）
 连续端口段：25:587（25-587之间的所有端口）" && echo
	fi
	read -e -p "(回车默认取消):" PORT
	[[ -z "${PORT}" ]] && echo "已取消..." && display_out_port && exit 0
}

# 出网关键词模块
disable_want_keyworld_out() {
	s="A"
	input_want_keyworld_type "ban"
	set_out_keywords
	echo -e "${Info} 已封禁关键词 [ ${key_word} ] !\n"
	while true; do
		input_want_keyworld_type "ban" "ban_1"
		set_out_keywords
		echo -e "${Info} 已封禁关键词 [ ${key_word} ] !\n"
	done
	display_out_keyworld
}
able_want_keyworld_out() {
	s="D"
	grep_out_keyword
	[[ -z ${disable_out_keyworld_list} ]] && echo -e "${Error} 检测到未封禁任何 关键词 !" && exit 0
	input_want_keyworld_type "unban"
	set_out_keywords
	echo -e "${Info} 已解封关键词 [ ${key_word} ] !\n"
	while true
	do
		grep_out_keyword
		[[ -z ${disable_out_keyworld_list} ]] && echo -e "${Error} 检测到未封禁任何 关键词 !" && exit 0
		input_want_keyworld_type "unban" "ban_1"
		set_out_keywords
		echo -e "${Info} 已解封关键词 [ ${key_word} ] !\n"
	done
	display_out_keyworld
}
able_all_keyworld_out() {
	grep_out_keyword
	[[ -z ${disable_out_keyworld_text} ]] && echo -e "${Error} 检测到未封禁任何 关键词，请检查 !" && exit 0
	if [[ ! -z "${v6iptables}" ]]; then
		Ban_KEY_WORDS_v6_num=$(echo -e "${disable_out_keyworld_v6_list}"|wc -l)
		for((integer = 1; integer <= ${Ban_KEY_WORDS_v6_num}; integer++))
			do
				${v6iptables} -t mangle -D OUTPUT 1
		done
	fi
	Ban_KEY_WORDS_num=$(echo -e "${disable_out_keyworld_list}"|wc -l)
	for((integer = 1; integer <= ${Ban_KEY_WORDS_num}; integer++))
		do
			${v4iptables} -t mangle -D OUTPUT 1
	done
	save_iptables_v4_v6
	display_out_keyworld
	echo -e "${Info} 已解封所有关键词 !"
}
input_want_keyworld_type() {
	Type=$1
	Type_1=$2
	if [[ $Type_1 != "ban_1" ]]; then
		echo -e "请选择输入类型：
 1. 手动输入（只支持单个关键词）
 2. 本地文件读取（支持批量读取关键词，每行一个关键词）
 3. 网络地址读取（支持批量读取关键词，每行一个关键词）" && echo
		read -e -p "(默认: 1. 手动输入):" key_word_type
	fi
	[[ -z "${key_word_type}" ]] && key_word_type="1"
	if [[ ${key_word_type} == "1" ]]; then
		if [[ $Type == "ban" ]]; then
			input_disable_want_keyworld
		else
			input_able_want_keyworld
		fi
	elif [[ ${key_word_type} == "2" ]]; then
		input_disable_keyworlds_file
	elif [[ ${key_word_type} == "3" ]]; then
		input_disable_keyworlds_url
	else
		if [[ $Type == "ban" ]]; then
			input_disable_want_keyworld
		else
			input_able_want_keyworld
		fi
	fi
}
input_disable_want_keyworld() {
	echo -e "请输入欲封禁的 关键词（域名等，仅支持单个关键词）"
	if [[ ${Type_1} != "ban_1" ]]; then
		echo -e "${Green_font_prefix}========示例说明========${Font_color_suffix}
 关键词：youtube，即禁止访问任何包含关键词 youtube 的域名。
 关键词：youtube.com，即禁止访问任何包含关键词 youtube.com 的域名（泛域名屏蔽）。
 关键词：www.youtube.com，即禁止访问任何包含关键词 www.youtube.com 的域名（子域名屏蔽）。
 更多效果自行测试（如关键词 .zip 即可禁止下载任何 .zip 后缀的文件）。" && echo
	fi
	read -e -p "(回车默认取消):" key_word
	[[ -z "${key_word}" ]] && echo "已取消..." && display_out_keyworld && exit 0
}
input_able_want_keyworld() {
	echo -e "请输入欲解封的 关键词（根据上面的列表输入完整准确的 关键词）" && echo
	read -e -p "(回车默认取消):" key_word
	[[ -z "${key_word}" ]] && echo "已取消..." && display_out_keyworld && exit 0
}
set_out_keywords() {
	key_word_num=$(echo -e "${key_word}" | wc -l)
	for ((integer = 1; integer <= ${key_word_num}; integer++)); do
		i=$(echo -e "${key_word}" | sed -n "${integer}p")
		out_keyworld_rule $v4iptables "$i" $s
		[[ ! -z "$v6iptables" ]] && out_keyworld_rule $v6iptables "$i" $s
	done
	save_iptables_v4_v6
}
out_keyworld_rule() {
	$1 -t mangle -$3 OUTPUT -m string --string "$2" --algo bm --to 65535 -j DROP
}
input_disable_keyworlds_file() {
	echo -e "请输入欲封禁/解封的 关键词本地文件（请使用绝对路径）" && echo
	read -e -p "(默认 读取脚本同目录下的 key_word.txt ):" key_word
	[[ -z "${key_word}" ]] && key_word="key_word.txt"
	if [[ -e "${key_word}" ]]; then
		key_word=$(cat "${key_word}")
		[[ -z ${key_word} ]] && echo -e "${Error} 文件内容为空 !" && View_ALL && exit 0
	else
		echo -e "${Error} 没有找到文件 ${key_word} !" && display_out_keyworld && exit 0
	fi	
}
input_disable_keyworlds_url() {
	echo -e "请输入欲封禁/解封的 关键词网络文件地址（例如 http://xxx.xx/key_word.txt）" && echo
	read -e -p "(回车默认取消):" key_word
	[[ -z "${key_word}" ]] && echo "已取消..." && View_ALL && exit 0
	key_word=$(wget --no-check-certificate -t3 -T5 -qO- "${key_word}")
	[[ -z ${key_word} ]] && echo -e "${Error} 网络文件内容为空或访问超时 !" && display_out_keyworld && exit 0
}

able_want_keyworld_out() {
	s="D"
	grep_out_keyword
	[[ -z ${disable_out_keyworld_list} ]] && echo -e "${Error} 检测到未封禁任何 关键词 !" && exit 0
	input_want_keyworld_type "unban"
	set_out_keywords
	echo -e "${Info} 已解封关键词 [ ${key_word} ] !\n"
	while true; do
		grep_out_keyword
		[[ -z ${disable_out_keyworld_list} ]] && echo -e "${Error} 检测到未封禁任何 关键词 !" && exit 0
		input_want_keyworld_type "unban" "ban_1"
		set_out_keywords
		echo -e "${Info} 已解封关键词 [ ${key_word} ] !\n"
	done
	view_all_disable_out
}

# 出网细分功能模块
# 封禁BT、PT、SPAM
disable_all_out() {
	disable_btpt
	disable_spam
}

disable_btpt() {
	check_BT
	[[ ! -z ${BT_KEY_WORDS} ]] && echo -e "${Error} 检测到已封禁BT、PT 关键词，无需再次封禁 !" && exit 0
	s="A"
	Set_BT
	echo -e "${Info} 已封禁BT、PT 关键词 !"
}
check_BT() {
	grep_out_keyword
	BT_KEY_WORDS=$(echo -e "$disable_out_keyworld_list" | grep "torrent")
}
Set_BT() {
	key_word=${bt_key_word}
	set_out_keywords
	save_iptables_v4_v6
}

disable_spam() {
	check_SPAM
	[[ ! -z ${SPAM_PORT} ]] && echo -e "${Error} 检测到已封禁SPAM(垃圾邮件) 端口，无需再次封禁 !" && exit 0
	s="A"
	Set_SPAM
	echo -e "${Info} 已封禁SPAM(垃圾邮件) 端口 !"
}
check_SPAM() {
	grep_out_port
	SPAM_PORT=$(echo -e "$disable_outport_list" | grep "${smtp_port}")
}
Set_SPAM() {
	if [[ -n "$v4iptables" ]] && [[ -n "$v6iptables" ]]; then
		Set_SPAM_Code_v4_v6
	elif [[ -n "$v4iptables" ]]; then
		Set_SPAM_Code_v4
	fi
	save_iptables_v4_v6
}
Set_SPAM_Code_v4() {
	for i in ${smtp_port} ${pop3_port} ${imap_port} ${other_port}; do
		tcp_outport_rules $v4iptables "$i" $s
		ucp_outport_rules $v4iptables "$i" $s
	done
}
Set_SPAM_Code_v4_v6() {
	for i in ${smtp_port} ${pop3_port} ${imap_port} ${other_port}; do
		for j in $v4iptables $v6iptables; do
			tcp_outport_rules $j "$i" $s
			udp_outport_rules $j "$i" $s
		done
	done
}

# 解封BT、PT、SPAM
able_all_out() {
	able_btpt
	able_spam
}

able_btpt() {
	check_BT
	[[ -z ${BT_KEY_WORDS} ]] && echo -e "${Error} 检测到未封禁BT、PT 关键词，请检查 !" && exit 0
	s="D"
	Set_BT
	echo -e "${Info} 已解封BT、PT 关键词 !"
}

able_spam() {
	check_SPAM
	[[ -z ${SPAM_PORT} ]] && echo -e "${Error} 检测到未封禁SPAM(垃圾邮件) 端口，请检查 !" && exit 0
	s="D"
	Set_SPAM
	view_all_disable_out
	echo -e "${Info} 已解封SPAM(垃圾邮件) 端口 !"
}

# 入网端口模块
able_want_port_in() {
	display_in_port
	s="A"
	input_able_want_inport
	set_in_ports
	echo -e "${Info} 已放行端口 [ ${PORT} ] !\n"
	able_port_Type_1="1"
	while true
	do
		input_able_want_inport
		set_in_ports
		echo -e "${Info} 已放行端口 [ ${PORT} ] !\n"
	done
}
input_able_want_inport(){
	echo -e "请输入欲放行的 入网端口（单端口/多端口/连续端口段）"
	if [[ ${able_port_Type_1} != "1" ]]; then
	echo -e "${Green_font_prefix}========入网端口示例说明========${Font_color_suffix}
 单端口：25（单个端口）
 多端口：25,26,465,587（多个端口用英文逗号分割）
 连续端口段：25:587（25-587之间的所有端口）" && echo
	fi
	read -e -p "(回车默认取消):" PORT
	[[ -z "${PORT}" ]] && echo "已取消..." && display_in_port && exit 0
}
disable_want_port_in(){
	display_in_port
	s="D"
	input_disable_want_inport
	set_in_ports
	echo -e "${Info} 已取消放行端口 [ ${PORT} ] !\n"
	able_port_Type_1="1"
	while true
	do
		input_disable_want_inport
		set_in_ports
		echo -e "${Info} 已取消放行端口 [ ${PORT} ] !\n"
	done
}
input_disable_want_inport(){
	echo -e "请输入欲取消的 入网端口（单端口/多端口/连续端口段）"
	if [[ ${able_port_Type_1} != "1" ]]; then
	echo -e "${Green_font_prefix}========入网端口示例说明========${Font_color_suffix}
 单端口：25（单个端口）
 多端口：25,26,465,587（多个端口用英文逗号分割）
 连续端口段：25:587（25-587之间的所有端口）" && echo
	fi
	read -e -p "(回车默认取消):" PORT
	[[ -z "${PORT}" ]] && echo "已取消..." && display_in_port && exit 0
}
set_in_ports() {
	if [[ -n "$v4iptables" ]] && [[ -n "$v6iptables" ]]; then
		tcp_inport_rules $v4iptables $PORT $s
		udp_inport_rules $v4iptables $PORT $s
		tcp_inport_rules $v6iptables $PORT $s
		udp_inport_rules $v6iptables $PORT $s
	elif [[ -n "$v4iptables" ]]; then
		tcp_inport_rules $v4iptables $PORT $s
		udp_inport_rules $v4iptables $PORT $s
	fi
	save_iptables_v4_v6
}
tcp_inport_rules() {
	[[ "$1" = "$v4iptables" ]] && $1 -t filter -$3 INPUT -p tcp -m multiport --dports "$2" -j ACCEPT -m comment --comment "shellsettcp"
	[[ "$1" = "$v6iptables" ]] && $1 -t filter -$3 INPUT -p tcp -m multiport --dports "$2" -j ACCEPT -m comment --comment "shellsettcp"
}
udp_inport_rules() {
	$1 -t filter -$3 INPUT -p udp -m multiport --dports "$2" -j ACCEPT -m comment --comment "shellsetudp";
}

# 入网IP模块
able_in_ips() {
	echo "设计中"
}
display_in_ip() {
	echo "设计中"
}

# 部分调用函数
save_iptables_v4_v6() {
	if [[ ${release} == "centos" ]]; then
		if [[ ! -z "$v6iptables" ]]; then
			service ip6tables save
			chkconfig --level 2345 ip6tables on
		fi
		service iptables save
		chkconfig --level 2345 iptables on
	else
		if [[ ! -z "$v6iptables" ]]; then
			ip6tables-save >/etc/ip6tables.up.rules
			echo -e "#!/bin/bash\n/sbin/iptables-restore < /etc/iptables.up.rules\n/sbin/ip6tables-restore < /etc/ip6tables.up.rules" >/etc/network/if-pre-up.d/iptables
		else
			echo -e "#!/bin/bash\n/sbin/iptables-restore < /etc/iptables.up.rules" >/etc/network/if-pre-up.d/iptables
		fi
		iptables-save >/etc/iptables.up.rules
		chmod +x /etc/network/if-pre-up.d/iptables
	fi
}

display_out_port() {
	grep_out_port
	echo -e "===============${Red_background_prefix} 当前已封禁 端口 ${Font_color_suffix}==============="
	echo -e "$disable_outport_list" && echo && echo -e "==============================================="
}

display_out_keyworld() {
	grep_out_keyword
	echo -e "==============${Red_background_prefix} 当前已封禁 关键词 ${Font_color_suffix}=============="
	echo -e "$disable_out_keyworld_list" && echo -e "==============================================="
}

display_in_port() {
	grep_tcp_inport
	grep_udp_inport
	if [[ -n ${able_tcp_inport_list} ]] || [[ -n ${able_udp_inport_list} ]]; then
	echo -e "===============${Red_background_prefix} 当前已放行 端口 ${Font_color_suffix}==============="
	fi
	if [[ -n ${able_tcp_inport_list} ]]; then
		echo
		echo "TCP"
		echo -e "$able_tcp_inport_list" && echo && echo -e "==============================================="
	fi
	if [[ -n ${able_udp_inport_list} ]]; then
		echo
		echo "UDP"
		echo -e "$able_udp_inport_list" && echo && echo -e "==============================================="
	fi
}

display_ssh() {
	get_ssh_port
	echo
	echo "SSH 端口为 ${PORT}"
	echo
}

get_ssh_port() {
	PORT=$(netstat -anp | grep sshd | awk 'NR==1{print  substr($4, 9, length($4)-8)}')
}
grep_out_port() {
	disable_outport_list=$(iptables -t filter -L OUTPUT -nvx --line-numbers | grep "REJECT" | awk '{print $13}')
}
grep_tcp_inport() {
	able_tcp_inport_list=$(iptables -t filter -L INPUT -nvx --line-numbers | grep "shellsettcp" | awk '{print $13}')
}
grep_udp_inport() {
	able_udp_inport_list=$(iptables -t filter -L INPUT -nvx --line-numbers | grep "shellsetudp" | awk '{print $13}')
}

grep_out_keyword() {
	disable_out_keyworld_list=""
	disable_out_keyworld_v6_list=""
	if [[ ! -z ${v6iptables} ]]; then
		disable_out_keyworld_v6_text=$(${v6iptables} -t mangle -L OUTPUT -nvx --line-numbers | grep "DROP")
		disable_out_keyworld_v6_list=$(echo -e "${disable_out_keyworld_v6_text}" | sed -r 's/.*\"(.+)\".*/\1/')
	fi
	disable_out_keyworld_text=$(${v4iptables} -t mangle -L OUTPUT -nvx --line-numbers | grep "DROP")
	disable_out_keyworld_list=$(echo -e "${disable_out_keyworld_text}" | sed -r 's/.*\"(.+)\".*/\1/')
}

diable_blocklist_out() {
	s="A"
	echo -e "正在连接 关键词网络文件地址"
	key_word=$(wget --no-check-certificate -t3 -T5 -qO- "https://raw.githubusercontent.com/Aipblock/iptaMshell/main/blocklists.txt")
	[[ -z ${key_word} ]] && echo -e "${Error} 网络文件内容为空或访问超时 !" && display_out_keyworld && exit 0
	key_word_num=$(echo -e "${key_word}"|wc -l)
	for((integer = 1; integer <= ${key_word_num}; integer++))
		do
			i=$(echo -e "${key_word}"|sed -n "${integer}p")
			set_out_keywords $v4iptables "$i" $s
			[[ ! -z "$v6iptables" ]] && set_out_keywords $v6iptables "$i" $s
	done
	save_iptables_v4_v6
	echo -e "成功执行" && echo
}

clear_rebuild_ipta() {
	rebuild_iptables_rule
	echo "已清空所有规则"
	able_ssh_port
	echo "仅放行了 SSH端口：${PORT}"
}

Update_Shell(){
	sh_new_ver=$(wget --no-check-certificate -qO- -t1 -T3 "https://raw.githubusercontent.com/Aipblock/iptaMshell/main/iptaMshell.sh"|grep 'sh_ver="'|awk -F "=" '{print $NF}'|sed 's/\"//g'|head -1)
	[[ -z ${sh_new_ver} ]] && echo -e "${Error} 无法链接到 Github !" && exit 0
	wget -N --no-check-certificate https://raw.githubusercontent.com/Aipblock/iptaMshell/main/iptaMshell.sh && chmod +x iptaMshell.sh && bash iptaMshell.sh
	echo -e "脚本已更新为最新版本[ ${sh_new_ver} ] !(注意：因为更新方式为直接覆盖当前运行的脚本，所以可能下面会提示一些报错，无视即可)" && exit 0
}

check_system
var_v4_v6_iptables
check_run
action=$1
if [[ ! -z $action ]]; then
	[[ $action = "banbt" ]] && Ban_BT && exit 0
	[[ $action = "banspam" ]] && Ban_SPAM && exit 0
	[[ $action = "banall" ]] && Ban_ALL && exit 0
	[[ $action = "unbanbt" ]] && UnBan_BT && exit 0
	[[ $action = "unbanspam" ]] && UnBan_SPAM && exit 0
	[[ $action = "unbanall" ]] && UnBan_ALL && exit 0
fi
echo && echo -e " iptables防火墙 管理脚本 ${Red_font_prefix}[v${sh_ver}]${Font_color_suffix}
  -- 基于逗比脚本修改 在此感谢大佬--
  -- 与某些转发管理面板可能会有冲突 --
————————————
  ${Red_font_prefix}出网方向功能
  ${Green_font_prefix}0.${Font_color_suffix} 查看 当前封禁列表
  ${Green_font_prefix}1.${Font_color_suffix} 封禁 BT、PT
  ${Green_font_prefix}2.${Font_color_suffix} 封禁 SPAM(垃圾邮件)
  ${Green_font_prefix}3.${Font_color_suffix} 封禁 BT、PT、SPAM
  ${Green_font_prefix}4.${Font_color_suffix} 封禁 自定义  端口
  ${Green_font_prefix}5.${Font_color_suffix} 封禁 自定义关键词
  ${Green_font_prefix}6.${Font_color_suffix} 解封 BT、PT
  ${Green_font_prefix}7.${Font_color_suffix} 解封 SPAM(垃圾邮件)
  ${Green_font_prefix}8.${Font_color_suffix} 解封 BT、PT+SPAM
  ${Green_font_prefix}9.${Font_color_suffix} 解封 自定义  端口
 ${Green_font_prefix}10.${Font_color_suffix} 解封 自定义关键词
 ${Green_font_prefix}11.${Font_color_suffix} 解封 所有  关键词
 ${Green_font_prefix}12.${Font_color_suffix} 封禁 Blocklists

————————————
 ${Red_font_prefix}入网方向功能

${Green_font_prefix}13.${Font_color_suffix} 查看 当前放行端口
${Green_font_prefix}14.${Font_color_suffix} 查看 当前放行IP

${Green_font_prefix}15.${Font_color_suffix} 放行 自定义  端口
${Green_font_prefix}16.${Font_color_suffix} 删除 已放行  端口
${Green_font_prefix}17.${Font_color_suffix} 放行 自定义 IP

————————————
${Red_font_prefix}增强功能

${Green_font_prefix}18.${Font_color_suffix} 查看 当前SSH端口
${Green_font_prefix}19.${Font_color_suffix} 夺回出入控制(清空所有规则)

————————————
${Green_font_prefix}20.${Font_color_suffix} 升级脚本
${Red_font_prefix}注意:${Font_color_suffix} 本脚本与某些转发管理面板可能会有冲突
————————————
" && echo
shell_run_tips
read -e -p " 请输入数字 [0-19]:" num
case "$num" in
	0)
	view_all_disable_out
	;;
	1)
	disable_btpt
	;;
	2)
	disable_spam
	;;
	3)
	disable_all_out
	;;
	4)
	disable_want_port_out
	;;
	5)
	disable_want_keyworld_out
	;;
	6)
	able_btpt
	;;
	7)
	able_spam
	;;
	8)
	able_all_out
	;;
	9)
	able_want_port_out
	;;
	10)
	able_want_keyworld_out
	;;
	11)
	able_all_keyworld_out
	;;
	12)
	diable_blocklist_out
	;;
    13)
    display_in_port
	;;
	14)
    display_in_ip
	;;
	15)
    able_want_port_in
	;;
	16)
    disable_want_port_in
	;;
	17)
    able_in_ips
	;;
	18)
    display_ssh
	;;
	19)
    clear_rebuild_ipta
	;;
	20)
    Update_Shell
	;;
	*)
	echo "请输入正确数字 [0-19]"
	;;
esac
