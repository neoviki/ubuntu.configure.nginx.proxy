#!/bin/bash
##################################################
#
#   Author  : Viki ( @ ) Vignesh Natarajan
#   Contact : vikiworks.io
#
##################################################

########## COMMON_CODE_BEGIN()   ########
CMD=""
DOMAIN=""
INCOMING=""
FORWARDING=""

CLEAN=0

ARG0=$0
ARG1=$1
ARG2=$2
ARG3=$3
ARG4=$4
ARG5=$5

os_support_check(){
    OS_SUPPORTED=0

    #Check Ubuntu 18.04 Support    
    cat /etc/lsb-release | grep 18.04 2> /dev/null 1> /dev/null
    if [ $? -eq 0 ]; then
        OS_SUPPORTED=1
    fi

    #Check Ubuntu 16.04 Support    
    cat /etc/lsb-release | grep 18.04 2> /dev/null 1> /dev/null
    if [ $? -eq 0 ]; then
        OS_SUPPORTED=1
    fi

    if [ $OS_SUPPORTED -eq 0 ]; then
	echo
	echo "Utility is not supported in this version of linux"
	echo
	exit 1
    fi

}


get_command(){
    if [ "$ARG0" == "sudo" ]; then
        CMD="$ARG1"
	DOMAIN="$ARG2"
	INCOMING="$ARG3"
	FORWARDING="$ARG4"
    else
        CMD="$ARG0"
	DOMAIN="$ARG1"
	INCOMING="$ARG2"
	FORWARDING="$ARG3"
    fi
}

usage_check(){
   	if [ -z "$FORWARDING" ]; then  
	    if [ "$ARG0" == "sudo" ]; then
	        echo "usage: $0 $1 <DOMAIN_NAME> <INCOMING_IP>:<INCOMING_PORT> <FORWARDING_IP>:<FORWARDING_PORT>"
	    else
	        echo "usage: $1 <DOMAIN_NAME> <INCOMING_IP>:<INCOMING_PORT> <FORWARDING_IP>:<FORWARDING_PORT>"
            fi
	    echo ""
	    echo "Note: Do not add www. in front of domain name"
	    echo ""
	    echo ""
	    echo "Example: google.com"
	    echo ""
	    echo ""
	    exit 1
        fi

	if [[ "$DOMAIN" =~ ^www.* ]]; then
	    echo ""
    	    echo "    error: DOMAIN name should not start with www"
	    echo ""
	    exit 1
	fi

}

extract_args(){
	FORWARDING_IP=`echo $FORWARDING | awk -F: '{print $1}'`
	FORWARDING_PORT=`echo $FORWARDING | awk -F: '{print $2}'`
	INCOMING_IP=`echo $INCOMING | awk -F: '{print $1}'`
	INCOMING_PORT=`echo $INCOMING | awk -F: '{print $2}'`
	error=0


	if [ -z $INCOMING_IP ]; then 
		echo "INCOMING_IP is empty"
		error=1
	fi

	if [ -z $INCOMING_PORT ]; then 
		echo "INCOMING_PORT is empty"
		error=1
	fi

	if [ -z $FORWARDING_IP ]; then
		echo "FORWARDING_IP is empty"
		error=1
	fi

	if [ -z $INCOMING_PORT ]; then
		echo "INCOMING_PORT is empty"
		error=1
	fi

	if [ $error -eq 1 ]; then 
		exit 1
	fi

	echo
	echo "    Domain             : $DOMAIN"
	echo
	echo "    Incoming IP        : $INCOMING_IP"
	echo "    Incoming PORT      : $INCOMING_PORT"
	echo
	echo "    Forwarding IP      : $FORWARDING_IP"
	echo "    Forwarding PORT    : $FORWARDING_PORT"
	echo
}


check_permission(){
    touch /bin/test.txt 2> /dev/null 1>/dev/null

    if [ $? -ne 0 ]; then
	echo "permission error, try to run this script wih sudo option"; 
	echo ""
	echo "Example: sudo $CMD"
	echo ""
	exit 1; 
    fi 
    
    rm /bin/test.txt
}

init_bash_installer(){
    os_support_check
    get_command
    check_permission
}
########## COMMON_CODE_END()   ########



generate_config(){
	rm -rf tmp.conf 2>/dev/null 1>/dev/null

cat > tmp.conf << EOF
server { 
	listen       ${INCOMING_PORT};
	server_name  ${DOMAIN} www.${DOMAIN};

	location /  {
		#If you want to forward to docker container, you can get the docker container ip address using
		#docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}'  <container id>

		#<FORWARDING_IP>:<FORWARDING_PORT>
		proxy_pass http://${FORWARDING_IP}:${FORWARDING_PORT};
		proxy_buffering off;

		#<Public Ip Address / Host Ip Address > : <Host Port for HTTP Traffic>
		#<INCOMING_IP>:<INCOMING_PORT>
		proxy_set_header X-Real-IP ${INCOMING_IP}:${INCOMING_PORT};
	}

}
EOF
    mv tmp.conf /etc/nginx/conf.d/${DOMAIN}.conf


    echo "NGINX Config File -> /etc/nginx/conf.d/${DOMAIN}.conf"
}

verify_config(){
    nginx -t
    [ $? -ne 0 ] && { echo "error line ( ${LINENO} )"; exit 1; }
}

reload_nginx(){
    service nginx reload
}

init_bash_installer
usage_check
extract_args
generate_config
verify_config
reload_nginx

