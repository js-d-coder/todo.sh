#!/bin/bash

# © 2018, Jibran Shaikh (https://github.com/js-d-coder/todo.sh), MIT License

# Keep track of your daily tasks, todo, reminders.

# USAGE:
# `OPERATION TASK [on date]'
# OPERATION can be add/delete/edit/view.
# TASK can be name of your tasks, todo, reminders.
# `on date' is optional; it adds TASK for the date given.
# TASK( or just task) without date appear in the list everyday i.e. they repeat everyday.
# With add operation you can set the name of the task and summary on it.
# With delete operation you can delete the task.
# With edit operation you can edit the summary of the task.
# With view operation you can view full summary of the task, 
# By default, only first two lines of the summary are shown in the list.

# When you launch the program you will see the task for the day and task
# that repeat everyday

# date can be any human readable date, man date for more.

############################ global variables ##########################

# today global variable of this script keeps the date of the day in which
# the program is run. Its format is YYMMDD, for example, 161230 implies
# 30 Dec 2016.
# This format is chosen because by default because if date command is given a 6
# digits long number, to convert to another form, it expects it to be
# of this form.

today=$(date +"%y%m%d")
tomorrow=$(date +"%y%m%d" -d tomorrow)

############################# coloured output #################################

# show_* functions are used to show coloured output.

function show_question {
echo -e "\033[1;34m$@\033[0m"
}

function show_info {
echo -e "\033[1;33m$@\033[0m"
}

function show_success {
echo -e "\033[1;32m$@\033[0m"
}

function show_error {
echo -e "\033[1;31m$@\033[m" 1>&2
}

################# strip beginning and ending whitespaces ######################

function stripWhiteSpace {
    echo "$*"
}

################### evalute the parameters ###################################

function evaluate {
    argc=$#
    argArray=( $@ )

    for ((i=$[ $# - 1 ]; i>=0; i--))
    do
        if [ "${argArray[$i]}" = "on"  ]
        then
            break
        fi
    done

    if [ $i -lt 1 ]
    then
        fileName="$*"	# name of the file name
		fileName=$(stripWhiteSpace $fileName)
        date=""	# date variabe will be empty
        return
    fi
    onPosition=$i # index of 'on' in argument array argArray
    for ((i=$[ $onPosition + 1 ]; i<${#argArray[@]}; i++))
    do
        checkDate="$checkDate ${argArray[$i]}"
    done

    # we have to checkDate variable. Check whether checkDate is a valid
    # date or not

    date=$(date +"%y%m%d" -d "$checkDate" 2>/dev/null )
    if [ -z "$date" ]
    then
        fileName="$*"
		fileName=$(stripWhiteSpace $fileName)
        date=""
    else
		for ((i=0;i<$onPosition;i++ ))
		do
			fileName="$fileName ${argArray[$i]}"
		done
		fileName=$(stripWhiteSpace $fileName)
		date=$(stripWhiteSpace $date)
    fi
}

############################ truncate history file ##########################
function truncateHistFile {
truncated=$(uniq .hist_file | tail -n 5)
echo "$truncated" > .hist_file
}

############################ set user preference #############################
function setPreference {
	if [ ! -r .todorc ]
	then
		read -p "Please set your prefered editor, EDITOR: " EDITOR
		echo "EDITOR=${EDITOR}" > .todorc
		HISTFILE=.hist_file
		echo "HISTFILE=.hist_file" >> .todorc
	fi
	source .todorc
	touch .hist_file
	truncateHistFile
	history -c
	history -r
}

################################### setting up ################################

# setUp function is first command that is run in this script.
# setUp function is used to check whether this script is being run for the first
# time by checking whether .todo directory exists or not. .todo directory
# which is created in user's home directory is used to store files needed by this
# script to work.

# .todo directory holds files that are named after TASK and sub-directories
# that are named after the particular date to hold the TASK for that date.
# Name of those directory are of the form YYMMDD.
# .todo and sub-directories contain a file named .count, which holds the
# names of TASK/files in that directory.

# If .todo directory does not exists is created. setUp function checks
# whether .todo directory exists or every time you run this script.

# record function (see below) is run once every time setUp function is run
# i.e. every time this script is run. record function is also run after
# some OPERATION (like add, delete). Read description of add and delete
# OPERATION for more.

function setUp {
tabs 4
clear
if ! cd ${HOME}
then
	show_error "${HOME} not defined";
	return 1
fi
if [ ! -d ${HOME}/.todo ]
then
	if mkdir -p ${HOME}/.todo 2>/dev/null # create .todo if it does not exists
	then
		show_info "Creating ${HOME}/.todo directory to store data"
	else
		show_error "Cannot create directory ${HOME}/.todo"
		return 1
	fi
fi
cd ${HOME}/.todo
if [ -r .lock ]
then
	if ! zenity --warning --text="Another instance of todo is running" &>/dev/null
	then
		show_info "Another instance of todo is running"
	fi
	exit
fi
touch .lock
setPreference
trap 'history -a;rm .lock;exit' 0 1 2 3 6
record $today
return 0
}

####################### make list of files or tasks ############################

# record function creates file named .count in .todo. This is file is used
# to keep a list of other files in .todo directory. Names of the files
# in .count file represents the names of TASK that repeat everyday

# record function can be passed with an argument of the form YYDDMM, which is
# date.

function record {

# takes a parameter, parameter should be a date of the form YYMMDD

# .count file keeps track of files in .todo directory.
# Each of these files keeps details of task/todo/reminder.

# Check to see if .count file already exists, if not create another one
find . -maxdepth 1 -type f ! -name 'archived' ! -name ".count" ! -name ".todorc" ! -name ".hist_file" ! -name ".lock"  -printf "%P\n" >.count

# check to see if .count file exists in directory keeping task/todo/reminder
# of particular day
if [ -d "$1" ]
then
    find ${1} -maxdepth 1 -type f ! -name 'archived' ! -name ".count" ! -name ".todorc" ! -name ".hist_file" ! -name ".lock" -printf "%P\n" > "${1}/.count"
fi
}

################################ command usage #################################

function usage {
show_info "\tUsage: ACTION TASK/TODO/REMINDER [on DATE]"
show_info "\t\tACTION can be add / view / delete / edit"
show_info "\t\tDATE can be any human readable date format (man date for details)"
show_info "\tusage examples:"
show_info "\t\tadd meet Mr. X on sunday # to add meeting with Mr. X on sunday"
show_info "\t\tview meet Mr. X on sunday # to see details about the event"
show_info "\t\tdelete meet Mr. X on 04/15/2016 # to cancel/delete the event"
show_info "\t\tedit meet Mr. X on sunday # to edit the details about the event"
}

##################### read user input #################################
function readInput {
read -er -p " ➜  " $1
history -s ${!1}
}

###################### handle user actions #####################################

function actions {
readInput INPUT
case $INPUT in
'add '* )
    $INPUT;;  ### run user's action ( add/view/delete/edit )
'view '* )
    $INPUT;;  ### run user's action ( add/view/delete/edit )
'delete '* )
    $INPUT;;  ### run user's action ( add/view/delete/edit )
'edit '* )
    $INPUT;;  ### run user's action ( add/view/delete/edit )
'?' )
    usage
    actions;;
"" )
    clear
    exit 0;;
* )
    show_error "Incorrect action, try again"
    actions;;
esac
}

################################### add ########################################

function add {

fileName="" # empty variable that will be holding file name
date="" # empty variable that would keep a valid date
checkDate="" # empty variable that will be holding date

evaluate $@

# if user wants to schedule task on date then date variable won't be empty
if [ -n "${date}" ]
then
    if [ -r "${date}/${fileName}" ]
    then
        clear
        show_error "'$fileName' on $(date +"%b %d %Y" -d "$date") already exists"
    else
        mkdir -p "$date"
        ${EDITOR} "${date}/${fileName}"
        record "$date"
        clear
        # if date command is given 6 digits number it expects it to be of the
        # form %YY%MM%DD
        if [ -f "${date}/${fileName}" ]
        then
            show_success "Added '$fileName' on $(date +"%b %d %Y" -d "$date")"
        fi
    fi
else
    if [ -r "${fileName}" ]
    then
        clear
        show_error "'$fileName' already exists"
    else
        ${EDITOR} "${fileName}"
        record "$date"
        clear
        if [ -f "${fileName}" ]
        then
            show_success "Added '$fileName'"
        fi
    fi
fi

}

################################### edit ######################################

function edit {

fileName="" # empty variable that will be holding file name
date="" # empty variable that would keep a valid date
checkDate="" # empty variable that will be holding date

evaluate $@

if [ -n "${date}" ]
then
    if [ -r "${date}/${fileName}" ]
    then
        ${EDITOR} "${date}/${fileName}"
        clear
        show_success "Edited '$fileName' on $(date +"%b %d %Y" -d "$date")"
    else
        clear
        show_error "'$fileName' on $(date +"%b %d %Y" -d "$date") does not exists"
    fi
else
    if [ -r "${fileName}" ]
    then
        ${EDITOR} "${fileName}"
        clear
        show_success "Edited '$fileName'"
	elif [ -r "${today}/${fileName}" ]
	then
		${EDITOR} "${today}/${fileName}"
		clear
		show_success "Edited '${fileName}'"
    else
        clear
        show_error "'$fileName' does not exists"
    fi
fi

}

################################### view ########################################

function view {

fileName="" # empty variable that will be holding file name
date="" # empty variable that would keep a valid date
checkDate="" # empty variable that will be holding date

evaluate $@

if [ -n "${date}" ]
then
    if [ -r "${date}/${fileName}" ]
    then
        less "${date}/${fileName}"
        clear
    else
        clear
        show_error "'$fileName' on $(date +"%b %d %Y" -d "$date") does not exists"
    fi
else
    if [ -r "${fileName}" ]
    then
        less "${fileName}"
        clear
	elif [ -r "${today}/${fileName}" ]
	then
		less "${today}/${fileName}"
		clear
    else
        clear
        show_error "'$fileName' does not exists"
    fi
fi

}

################################### delete ########################################

function delete {

fileName="" # empty variable that will be holding file name
date="" # empty variable that would keep a valid date
checkDate="" # empty variable that will be holding date

evaluate $@

if [ -n "${date}" ]
then
    if [ -r "${date}/${fileName}" ]
    then
        rm "${date}/${fileName}"
        record "$date"
        clear
        show_success "Deleted '$fileName' on $(date +"%b %d %Y" -d "$date")"
    else
        clear
        show_error "'$fileName' on $(date +"%b %d %Y" -d "$date") does not exists"
    fi
else
    if [ -r "${fileName}" ]
    then
        rm "${fileName}"
        record "$date"
        clear
        show_success "Deleted '$fileName'"
	elif [ -r "${today}/${fileName}" ]
	then
		rm "${today}/${fileName}"
		record "${today}"
		clear
		show_success "Deleted '${fileName}'"
    else
        clear
        show_error "'$fileName' does not exists"
    fi
fi

}

############################# greet the user ###############################

function greetings {
time=$(date +"%H%M")
if [ $time -lt 1200 ]
then
show_info "Good morning, $USER"
elif [ $time -lt 1700 ]
then
show_info "Good afternoon, $USER"
else
show_info "Good evening, $USER"
fi
}

############################## list today's tasks ##########################

function listTasks {
if [ -s .count ] || [ -s "${today}/.count" ] || [ -s "${tomorrow}/.count" ]
then
    if  [ -s .count ] || [ -s "${today}/.count" ]
    then
        echo
        show_info	"#----------------------#"
        show_info	"#  Today's tasks are   #"
        show_info	"#----------------------#"
        echo
        if [ -s "${today}/.count" ] ### list today's task/todo/reminder
        then
            IFSOLD="$IFS"
            IFS=$'\n'
            for line in $(cat "${today}/.count")
            do
                echo -en "\t# ${line}"
                show_info " due today"
                lineLimit=0 ### this variable is used to limit the number of lines of
                           ### content of the task/todo/reminder to show to 2
                for content in $(cat "${today}/${line}")
                do
                    let lineLimit++
                    if [ $lineLimit -gt 1 ]
                    then
                        echo -e "\t\t..."
                        break
                    else
                        echo -e "\t\t${content:0:77}..."
                    fi
                done
                # echo
            done
            IFS="$IFSOLD"
        fi
        if [ -s .count ] # list repeated task/todo/reminder
        then
            IFSOLD="$IFS"
            IFS=$'\n'
            for line in $(cat .count)
            do
                echo -e "\t# $line"
                # this variable is used to limit the number of lines of content
                # of the task/todo/reminder show to 2.
                lineLimit=0
                for content in $(cat "${line}")
                do
                    let lineLimit++
                    if [ $lineLimit -gt 1 ]
                    then
                        echo -e "\t\t..."
                        break
                    else
                        echo -ne "\t\t${content:0:77}"
						if [ ${#content} -gt 77 ]
						then
							echo "..."
						else
							echo ""
						fi
                    fi
                done
                # echo
            done
            IFS="$IFSOLD"
        fi
    fi
    if [ -s "${tomorrow}/.count" ]
    then
        if [ -s "${tomorrow}/.count" ]
        then
            echo
            show_info	"#-------------------------#"
            show_info	"#  Tomorrow's tasks are   #"
            show_info	"#-------------------------#"
            echo
            IFSOLD="$IFS"
            IFS=$'\n'
            for line in $(cat "${tomorrow}/.count")
            do
                echo -e "\t# $line"
            done
        IFS="$IFSOLD"
        fi
    fi
    echo
    show_question "What would you like to do?"
   #echo -e "\tadd\t\tTASK/TODO/REMINDER [ on date]"
   # echo -e "\tedit\tTASK/TODO/REMINDER [ on date]"
   # echo -e "\tview\tTASK/TODO/REMINDER [ on date]"
   # echo -e "\tdelete\tTASK/TODO/REMINDER [ on date]"
    echo -e "\tEnter ? for help"
    echo -e "\tPress Enter key or ^D to exit"
else
    echo
    show_error "You don't have any task/todo/reminder. You can start by adding one."
    echo
    show_question "What would you like to do?"
    echo
    echo -e "\tType 'add TASK/TODO/REMINDER' to add your task or todo or reminder"
    echo -e "\tType ? for usage and examples"
    echo -e	"\tPress Enter or ^D to exit\n"
fi
}

###############################################################################

if ! setUp # stop if setup fails
then
    read -er
    exit 1
fi
greetings
while echo >/dev/null
do
    listTasks
    actions
done
