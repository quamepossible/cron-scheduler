#!/bin/bash

# Name: Kwame Opoku - Appiah
# Program: Custom cronjob scheduler application 
# Description: This is a simple script to allow the creation of a cronjob interactively from the command line
              # This program also allows the modification and deletion of existing cronjobs, all from the command line
	      #
	      # Features: 1. Add cronjobs
	      # 	  2. Modify cronjobs
	      # 	  3. Delete cronjobs
	      # 	  4. Input validations
	      # 	  5. When adding a cronjob, only valid time values are accepted,
	      # 	     program checks for invalid time range, and alerts user

# disable globbing		  
set -f

SUBMITTED=0
hold_jobs=()


# create a file called 'cron-tasks' to store cronjobs if it doesn't exist
#
if [ ! -f "./cron-tasks" ]; then
	touch cron-tasks
fi

### read_jobs() reads existing cronjobs and store them in an array 'hold_jobs=()'
read_jobs(){
	while read job; do
		hold_jobs+=("$job")
	done < cron-tasks 
}

### read cronjobs upon program execution
read_jobs

### this array stores the timer values (mins hrs days months weekday) for every cronjob
hold_timer=()


### modify_delete_fnc() handles the modification and deletion of cronjobs

modify_delete_fnc(){
	# check if there are any cronjobs to modify or delete 
	if [ ${#hold_jobs[@]} -eq 0 ]; then
		echo
		echo "-------------------------------"
		echo "| No cronjob found in crontab |"
		echo "|                             |"
		echo "| Add at least 1 cron job     |"
		echo "| to 'Modify' or 'Delete'     |"
		echo "-------------------------------"
		echo
		exit 1
	fi
	echo
	echo "///////////////////////////////////////////"
	echo "/"
	echo "/ Usage: Enter line number to $1"
	echo "/"
	echo "/ Eg: Enter '1' to ${1}: ${hold_jobs[0]}"
	echo "/"
	echo "////////////////////////////////////////////"
	echo
	echo
	echo "----------   list of all cronjobs   ---------"
	echo
	cron_count=1
	while read job; do
		echo "${cron_count}. ${job}"
		((cron_count++))
	done < cron-tasks
	echo
	echo "---------------   end of list   --------------"
	echo
	echo -n "Enter Line Number: "	
	read modify_num
	if [[ -z "$modify_num" ]]; then
		echo 
		echo "------------------------------------------"
		echo "| Line Number to ${1} is required "
		echo "------------------------------------------"
		echo
		exit 1
	fi

	# check if user entered a number in range (1-n), for which line to modify or delete
	if [[ "$modify_num" =~ ^[1-9]+$ ]]; then

		max_job_line=$(wc -l ./cron-tasks | cut -d' ' -f1) # get number of lines (or cronjobs) in crontab
		max_job_line=$((max_job_line)) # explicitly convert value to an integer

		num_line=$((modify_num)) # explicitly convert entered line number to integer

		# if line Number to 'Modify' or 'Delete' is less than or equal to total number of lines in crontab
		if [ $num_line -le $max_job_line ]; then

			# get the first argument '${1}' when function 'modify_delete_fnc' is called
			if [ "${1}" == "Modify" ]; then
				add_job "modify" "${num_line}"
			elif [ "${1}" == "Delete" ]; then
				echo
				echo -n "* Do you want to Delete cronjob on line ${num_line}? (Y/n) : "
				read del_response

				# if user response is 'empty' or Y or y or YES
				if [[ -z "$del_response" || $del_response =~ ^([Yy]|[Yy][Ee][Ss])$ ]]; then

					# use 'sed' to delete the specified Line from the crontab
					sed -i "${num_line}d" ./cron-tasks

					# apply changes
					crontab ./cron-tasks &> /dev/null
					echo
					echo "-------------------------------------------"
					echo "| Cronjob on line ${num_line} Deleted successfully "
					echo "-------------------------------------------"
					echo
				else
					echo
					echo "-----------------------------"
					echo "| Delete operation canceled |"
					echo "-----------------------------"
					echo
				fi
				exit 1
			fi
		else
			echo
			echo "*** Line Number does not exist ***"
			echo
		fi
	else
		echo
		echo "------------------------------------------------------"
		echo "| Line Number must be an Integer, and greater than 0 |"
		echo "------------------------------------------------------"
		echo
	fi
}


############ THIS FUNCTION VALIDATES USER INPUT FOR VALID CRON TIME RANGE ############
#
validate_timer(){
	time=$1
	range_min=$2
	range_max=$3
	info_one=$4
	info_two=$5
	pos=$6
	
	# check if cronjob runs at specific 'time' interval
	if [ "$time" == "*" ]; then
		hold_timer[$pos]="*"

	
	# if entered time value is in the range '*/n', where n is maximum range value
	#
	# eg: When adding time for minutes, valid value must be in the range of (*/0 - */59)
	elif [[ "$time" =~ ^[\*][\/][0-9]+$ ]]; then

		# get the 'n' value from */n 
		get_time=$(echo "$time" | awk -F/ '{print $NF}')	

		# explicitly convert value to integer
		get_time=$((get_time))


		# entered time value must be in a valid range of time type
		#
		# eg: for hours, n of */n must be in the range of (0 - 23)
		#     for months, n of */n must be in the range of (1 - 12)
		
		if [[ $get_time -lt ${range_min} || $get_time -gt ${range_max} ]]; then
			echo
			echo "--------------------------------"
			echo "| Error: ${info_two}" 
			echo "--------------------------------"
			echo
		else
			hold_timer[$pos]=$time
		fi


	# if entered time value is an integer from 0 - (maximum range value)
	#
	# if user entered 40 at minutes field, it will pass check, 
	  # because minutes field expects a value from 0 - 59
	  #
	elif [[ "$time" =~ ^[0-9]+$ ]]; then
		get_time=$((time))
		if [[ $get_time -lt ${range_min} || $get_time -gt ${range_max} ]]; then
			echo
			echo "------------------------------"
			echo "| Error: ${info_two} "
			echo "------------------------------"
			echo
		else
			hold_timer[$pos]=$time
		fi


	# this section runs if user entered an invalid time range 
	  # for mins or hrs or days or months or weekday
	else
		echo
		echo "----------------------------------------------"
		echo "| Invalid cron time set for ${info_one}: '$time' "
		echo "----------------------------------------------"
		echo
	fi	
}


add_job(){
	# reset variables to defaults
	hold_timer=()
	
	# accepts minutes value
	echo 
	echo -n "*** Enter Minutes (0-59): " 
	read mins
	echo

	# accepts hours value
	echo -n "*** Enter Hours (0-23): " 
	read hrs
	echo

	# accepts days value
	echo -n "*** Enter Day of month (1-31): " 
	read day
	echo 
	
	# accepts months value
	echo -n "*** Enter Month (1-12): " 
	read month
	echo
	
	# accepts weekdays value
	echo -n "*** Enter Weekday (0-6) (Sunday=0 or 7): "
	read weekday
	echo
	echo

	# accept path to script or command to execute
	echo -n "Enter path to an executable command or script: "
	read command_file

	if [[ -z "$mins" || -z "$hrs" || -z "$day" || -z "$month" || -z "$weekday" || -z "$command_file" ]]; then
		echo
		echo "------------------------------------"
		echo "| All cron timer field is required |"
		echo "------------------------------------"
		echo
		exit 1
	fi
	if [ ! -f "$command_file" ]; then
		echo
		echo "-----------------------------"
		echo "| Executable file not found |"
		echo "-----------------------------"
		echo
		exit 1
	fi

	set_times=("$mins" "$hrs" "$day" "$month" "$weekday")	
	time_counter=0
	for time in ${set_times[@]}; do
		if [ ${time_counter} -eq 0 ]; then
			validate_timer "$time" 0 59 "Minutes" "Invalid Minutes value; valid values: (0-59) or * or */n" $time_counter
		elif [ ${time_counter} -eq 1 ]; then
			validate_timer "$time" 0 23 "Hours" "Invalid Hour value; valid values: (0-23) or * or */n" $time_counter
		elif [ ${time_counter} -eq 2 ]; then
			validate_timer "$time" 1 31 "Days" "Invalid Day value; valid values: (1-31) or * or */n" $time_counter
		elif [ ${time_counter} -eq 3 ]; then
			validate_timer "$time" 1 12 "Month" "Invalid Month value; valid values: (1-12) or * or */n" $time_counter
		elif [ ${time_counter} -eq 4 ]; then
			validate_timer "$time" 0 7 "Weekday" "Invalid Weekday value; valid values: (0-7) or * or */n" $time_counter
		fi
		((time_counter++))
	done	

	
	# if hold_timer length = 5, it means all 5 (time) fields were received and valid
	#
	if [[ ${#hold_timer[@]} -eq 5 ]]; then

		# append command to timers
		hold_timer[5]="${command_file}"

		echo
		echo -n "* Do you want to SUBMIT cronjob (Y/n): "
		read submit
		
		if [[ -z "$submit" || "$submit" =~ ^([Yy]|[Yy][Ee][Ss])$ ]]; then
			SUBMITTED=1
			
			if [[ -n "$1" && "$1" == 'modify' ]]; then # we are modifying a line, instead of adding a new one

				# replace line to modify with new cronjob
				awk -v n="$2" -v new_job="${hold_timer[*]}" 'NR==n{$0=new_job}1' ./cron-tasks > temp && mv temp ./cron-tasks

				if [ $? -eq 0 ]; then
					echo
					echo "---------------------------------"
					echo "| Successfully Modified Line $2 |"
					echo "---------------------------------"
					echo
				else
					echo
					echo "----------------------------"
					echo "| Failed to Modify Line $2 |"
					echo "----------------------------"
					echo
					exit 1
				fi

			else
				echo
				echo "------------------------------"
				echo "| Successfully added cronjob |"
				echo "------------------------------"
				echo ${hold_timer[@]} >> ./cron-tasks
			fi
			crontab ./cron-tasks

			echo
		else
			echo
			echo "-------------------------------"
			echo "| Cronjob submission canceled |"
			echo "-------------------------------"
			echo
			exit 1
		fi
	else
		echo 
		echo "-------------------------"
		echo "| Adding cronjob failed |"
		echo "-------------------------"
		echo
		exit 1
	fi

	if [ $SUBMITTED -eq 1 ]; then
		echo 
		echo -n "* Do you want to add another cronjob (Y/n) ?: "
		read new_job
		if [[ "$new_job" =~ ^([Yy]|[Yy][Ee][Ss])$ ]]; then
			SUBMITTED=0
			add_job
		else
			echo
			echo "*** Program exited succesfully ***"
			echo
		fi
	fi

}

################ CODE EXECUTION STARTS HERE ################ 
echo
echo "Select action"
echo "A: (Add),  M: (Modify), D: (Delete)"
echo
echo -n "Enter Action: "
read action

if [[ -z "$action" ]]; then
	echo 
	echo "*** Action field required ***"
	echo
	exit 1
fi

# THIS SECTION ADDS A NEW CRONJOB
if [[ "$action" =~ ^[Aa]$ ]]; then
	add_job

# THIS SECTION MODIFIES AN EXISTING CRONJOB
elif [[ "$action" =~ ^[Mm]$ ]]; then
	modify_delete_fnc "Modify"	

# THIS SECTION DELETES A CRONJOB
elif [[ "$action" =~ ^[Dd]$ ]]; then
	echo
	modify_delete_fnc "Delete"

else
	echo 
	echo "----------------------------------"
	echo "| Invalid action value           |"
	echo "|                                |"
	echo "| Usage: A: (Add)                |"
	echo "|        M: (Modify)             |"
	echo "|        D: (Delete)             |"
	echo "----------------------------------"
	echo
fi

# disable globbing		  
set +f
exit 0

