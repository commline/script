#!/bin/bash

domain=$1
dkfolder="/etc/opendkim/"

display_usage() { 
	echo "This script must be run with super-user privileges." 
	echo -e "\nUsage:\nTo add DKIM for domain: $0 domain.com add\nTo add DKIM for domain: $0 domain.com remove\n" 
} 

# if less than two arguments supplied, display usage 
if [  $# -ne 2 ]; then 
	display_usage
	exit 1
fi

# check whether user had supplied -h or --help . If yes display usage 
if [[ ( $# == "--help") ||  $# == "-h" ]]; then 
	display_usage
	exit 1
fi 
 
# display usage if the script is not run as root user 
if [[ $USER != "root" ]]; then 
	echo "This script must be run as root!" 
	exit 1
fi 

remove_domain() {
	if ( ! grep -q $domain $dkfolder/KeyTable ); then
		echo "Domain $domain has no DKIM set"
		exit 1
	fi
	if ( grep -q $domain $dkfolder/KeyTable ); then
		sed -i "/$domain/D" $dkfolder/KeyTable
		sed -i "/$domain/D" $dkfolder/SigningTable
		rm -rf $dkfolder/keys/$domain 
	fi
        if ( ! grep -q $domain $dkfolder/KeyTable ); then
                echo "DKIM for $domain deleted"
		/etc/init.d/postfix restart
		/etc/init.d/opendkim restart
                exit 1
        fi
}

add_domain() {
	if ( grep -q $domain $dkfolder/KeyTable ); then
		echo "$domain already has configured DKIM"
		exit 1
	fi

	if [ ! -d $dkfolder/keys/$domain ]; then
		echo "DKIM keys folder not found, creating it."
		mkdir -p /etc/opendkim/keys/$domain
	fi

	echo "Generating DKIM keys"
	if ( ! /usr/bin/opendkim-genkey -D $dkfolder/keys/$domain/ -s mail -d $domain ); then
		echo "Unable to generate DKIM keys for domain $domain"
		exit 1
	fi

	echo "Adding record for $domain to $dkfolder/KeyTable"
	if ( ! echo -e "key.for.$domain	$domain:mail:$dkfolder/keys/$domain/mail.private" >> $dkfolder/KeyTable ); then
		echo "Unable to add record for $domain to $dkfolder/KeyTable"
		exit 1
	fi

	echo "Adding record for $domain to $dkfolder/SigningTable"
	if ( ! echo -e "*@$domain key.for.$domain" >> $dkfolder/SigningTable ); then
		echo "Unable to add record for $domain to $dkfolder/SigningTable"
		exit 1
	fi
	chown opendkim:postfix $dkfolder -R
	echo -e "HERE Is YOUR DNS TXT DKIM KEY/RECORD\n\n"
	cat $dkfolder/keys/$domain/mail.txt
	echo -e "\n\n"
}


if [[ $2 == "add" ]]; then
	add_domain;
fi
if [[ $2 == "remove" ]]; then
	remove_domain;
fi
