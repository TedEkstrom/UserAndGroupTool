#! /bin/bash

# Orgianalkod skriven av wafsek, Baljit Singh Sarai
# Hämtad ifrån https://gist.github.com/wafsek/b78cb3214787a605a28b

# Modifierad av Ted Ekström, 231024

clear

ggroup=""

function createUser () 
{
	user=$(whiptail --inputbox "Create new local user (using useradd --create-home --no-user-group)." 8 39 --title "Create new local user" 3>&1 1>&2 2>&3)
		
	if [[ $user ]] ; then
		#sudo -k  
		gsudo=$(whiptail --passwordbox "Enter you sudo password?" 8 39 $gsudo --title "Enter sudo password" 3>&1 1>&2 2>&3)
		echo $gsudo | sudo -S useradd $user --create-home --no-user-group
		whiptail --msgbox "User $user was created." 20 78
	else
		whiptail --msgbox "Something happend. Username was missing." 20 78
	fi
}


function addUser ()
{
		
	allgroups=$(cat /etc/group | awk '{sub(/:.*/,""); print}')
	counter=0
	grouplist=""
	for group in $allgroups ; do
		a="........."
		grouplist="$grouplist $group $a off " 	
		let counter=$counter+1
	done
	#Lägg till sökning av grupp 
	ggroup=$(
		whiptail --title "Available groups" --radiolist \
		"Chosse a group to edit." 20 78 13 \
		$grouplist 3>&2 2>&1 1>&3
	)

	local=$(
		whiptail --title "Local or domainuser" --radiolist \
		"Choose type av user account." 20 78 13 \
		"Local"  "........." off \
		"Domain" "........." off 3>&2 2>&1 1>&3
	)


	if [[ $local = "Local" ]] ; then
		if [[ $ggroup ]] ; then

			users=$(cat /etc/passwd | awk '{sub(/:.*/,""); print}')
			list=""
		
			for user in $users ; do
				a="........."
				list="$list $user $a off " 
				let counter=$counter+1
			done

			user=$(
 	        		whiptail --title "Select user to add." \
				--checklist "Select user to add" 0 0 $counter \
				$list 3>&2 2>&1 1>&3
			)
			
			user=$(echo $user | sed -e 's/^"//' -e 's/"$//')

			if [[ $user ]] ; then
				#sudo -k  
				gsudo=$(whiptail --passwordbox "Enter you sudo password?" 8 39 $gsudo --title "Enter sudo password" 3>&1 1>&2 2>&3)
				echo $gsudo | sudo -S usermod -a -G $ggroup $user
				whiptail --msgbox "Following user was added to group $group: \n$user" 20 78
			fi
		else
			whiptail --msgbox "Somting happend. No group was selected." 20 78
		fi
	else
		if [[ $ggroup ]] ; then

			user=$(whiptail --inputbox "Add user to $ggroup group (user@example.com)" 8 39  "user@example.com" --title "Add user to" 3>&1 1>&2 2>&3)

			if [[ $user ]] ; then
				#sudo -k  
				gsudo=$(whiptail --passwordbox "Enter you sudo password?" 8 39 $gsudo --title "Enter sudo password" 3>&1 1>&2 2>&3)
				#echo $gsudo | sudo -S useradd $user --create-home --no-user-group
				echo $gsudo | sudo -S sed -i "/^$ggroup:/ s/$/,$user/" /etc/group
				whiptail --msgbox "Following user was added to group $group: \n$user" 20 78
			fi
		else
			whiptail --msgbox "Somting happend. No group was selected." 20 78
		fi
	fi
}

function removeUser ()
{
		
	allgroups=$(cat /etc/group | awk '{sub(/:.*/,""); print}')
	grouplist=""
	for group in $allgroups ; do
		a="........."
		grouplist="$grouplist $group $a off " 	
	done
	#Lägg till sökning av grupp
	ggroup=$(
		whiptail --title "Available groups" --radiolist \
		"Chosse a group to edit." 20 78 13 \
		$grouplist 3>&2 2>&1 1>&3
	)

	counter=0
	if [[ $ggroup ]] ; then
		users=$(getent group $ggroup | awk '{ sub(/.*:/, ""); sub(/END:.*/, ""); print }' | tr "," "\n")
		list=""
		
		for user in $users ; do
			a="........."
			list="$list $user $a off " 
			let counter=$counter+1
		done

		CHOICE=$(
 	        	whiptail --title "Remove users from group $ggroup"
			--checklist "Choose user's to remove" 0 0 $counter \
			$list 3>&2 2>&1 1>&3
		)

		CHOICE=$(echo $CHOICE | sed -e 's/^"//' -e 's/"$//')

		if [[ $CHOICE ]] ; then
			#sudo -k  
			gsudo=$(whiptail --passwordbox "Enter you sudo password?" 8 39 $gsudo --title "Enter sudo password" 3>&1 1>&2 2>&3)
			echo $gsudo | sudo -S gpasswd -d $CHOICE docker
			whiptail --msgbox "Following user was removed from group $group: \n$users" 20 78
		fi
	else
		whiptail --msgbox "Something happend. No user vas select to be addded." 20 78
	fi
}

function checkUser () 
{
	allgroups=$(cat /etc/group | awk '{sub(/:.*/,""); print}')
	
	grouplist=""
	for group in $allgroups ; do
		a="..................................................."
		grouplist="$grouplist $group $a off " 	
	
	done
	#Lägg till sökning av grupp
	ggroup=$(
		whiptail --title "Available groups" --radiolist \
		"Chosse a group to check." 20 78 13 \
		$grouplist 3>&2 2>&1 1>&3
	)

	if [[ $ggroup ]] ; then
		
		users=$(getent group $ggroup | awk '{ sub(/.*:/, ""); sub(/END:.*/, ""); print }' | tr "," "\n")
		list=""
		for user in $users ; do
			list="$list $user" 
		done
		whiptail --msgbox "Users in group $group: \n$users" 20 78
		
	fi
}




function createGroup () 
{

	ggroup=$(whiptail --inputbox "Create new local group." 8 39 --title "Create new local user" 3>&1 1>&2 2>&3)
		
	if [[ $ggroup ]] ; then
	#	#sudo -k
		gsudo=$(whiptail --passwordbox "Enter you sudo password?" 8 39 $gsudo --title "Enter sudo password" 3>&1 1>&2 2>&3)
		echo $gsudo | sudo -S addgroup $ggroup
		whiptail --msgbox "Group $ggroup was created." 20 78
	else
		whiptail --msgbox "Something happend. Group was missing." 20 78
	fi
}

function removeGroup () 
{
	
	allgroups=$(cat /etc/group | awk '{sub(/:.*/,""); print}')
	grouplist=""
	for group in $allgroups ; do
		a="........."
		grouplist="$grouplist $group $a off " 	
	done
	#Lägg till sökning av grupp
	ggroup=$(
		whiptail --title "Available groups" --radiolist \
		"Chosse a group to edit." 20 78 13 \
		$grouplist 3>&2 2>&1 1>&3
	)

	if [[ $ggroup ]] ; then
	#	#sudo -k
		gsudo=$(whiptail --passwordbox "Enter you sudo password?" 8 39 $gsudo --title "Enter sudo password" 3>&1 1>&2 2>&3)
		echo $gsudo | sudo -S groupdel $ggroup
		whiptail --msgbox "Group $ggroup was created." 20 78
	else
		whiptail --msgbox "Something happend. Group was missing." 20 78
	fi
}


while [ 1 ]
do
CHOICE=$(
whiptail --title "User and group tool" --menu "Make your choice" 16 100 9 \
	"1)" "Create user"   \ 
	"2)" "Add users"    \ 
	"3)" "Remove users"  \
	"4)" "Check user access" \
	"5)" "Create group" \
	"6)" "Remove group" \
	"9)" "Quit"  3>&2 2>&1 1>&3	
)

case $CHOICE in
	"1)")   
		
		createUser
	;;
	"2)")   
		removeUser
	;;

	"3)")   
		checkUser
	;;

	"4)")   

                addUser
        ;;

	"5)")   

                createGroup
        ;;

	"6)")   

                removeGroup
        ;;
	"9)") exit
	;;
	"") exit
        ;;
esac
done
exit
