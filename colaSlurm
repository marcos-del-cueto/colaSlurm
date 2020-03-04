#!/bin/bash
#####################################################################
# Marcos del Cueto
#####################################################################
# -- Simple script to get info about slurm queue system
#
# -- Quickly shows info about user's jobs (-u), info about specific
#    partitions (-p) and general usage of slurm system (-s), with 
#    style based on savail by A S Muzas
#
# -- Script is home-made and is certainly not optimized. Feel free to
#    share any improvements
#
# -- Can be executed simply with: $> ./colaSlurm
#
# -- For convenience, can be moved to a directory that belongs to the user's path
#
# -- Relies on 'sinfo' and 'squeue' slurm commands
#
# -- May 2018
#####################################################################
# -- Updated March 2020
#####################################################################

#########################
#### START FUNCTIONS ####
#########################
# Prints summary
function print_summary ()
{
header="\n %10s %10s %10s %10s %10s %20s %11s"
format="%10s %10i %10i %10i %10i %15i %18s \n"
    printf "${header}" "PARTITION" "TOT CPUS" "AV CPUS" "JOBS R" "PD JOBS" "PD REQUESTED CPUS" "TIMEWALL"
    printf "${header}\n" "=========" "========" "=======" "======" "=======" "=================" "========"
    for j in `sinfo -o "%10P" | grep -v "PARTITION"`
    do
        PARTITION="${j}"
        timewall=`sinfo -o "%10P %15l" | awk -v part=${PARTITION} '$1 == part {print $2}'`  
        total_cpu=`sinfo -o "%10P %15C" | awk -v part=${PARTITION} '$1 == part {print $2}' | cut -f 4 -d "/"`
        running_jobs=`squeue -o "%7P %7C %7t" | awk -v part=${PARTITION} '{if(($1 == part)&&($3 == "R")) print $3}' | wc -l`
        available=`sinfo -o "%10P %15C" | awk -v part=${PARTITION} '$1 == part {print $2}' | cut -f 2 -d "/"`
        pending_jobs=`squeue -o "%7P %7C %7t" | awk -v part=${PARTITION} '{if(($1 == part)&&($3 == "PD")) print $3}' | wc -l`
        pending_CPUs=`squeue -o "%7P %7C %7t" | awk -v part=${PARTITION} '{if(($1 == part)&&($3 == "PD")) print $2}' | awk '{sum += $1} END {print sum}'`
        if [ "${pending_CPUs}" == "" ]
        then
             pending_CPUs=0
        fi
        printf "${format}" ${PARTITION} ${total_cpu} ${available} ${running_jobs} ${pending_jobs} ${pending_CPUs} ${timewall}
    done
    exit
}
# Prints specific user info
function print_user ()
{
    #squeue -u ${1} -o "%9i %5C %10q %10u %15a %15j %.4t %.13M %.8D %17R %22S %10p %6m"
    squeue -u ${1} -o "%15i %5C %5D %13P %10u %15j %.4t %.10M %.17R %10Q"
    user_jobs_running=`squeue -h -u ${1} -t R -o "%C" | wc -l`
    user_cpus_running=`squeue -h -u ${1} -t R -o "%C" | awk 'BEGIN{count=0}{count=count+$1}END{print count}'`
    user_jobs_pending=`squeue -h -u ${1} -t PD -o "%C" | wc -l`
    user_cpus_pending=`squeue -h -u ${1} -t PD -o "%C" | awk 'BEGIN{count=0}{count=count+$1}END{print count}'`
    echo "-------------------"
    echo "USER: ${1}"
    echo "Jobs Running: ${user_jobs_running}"
    echo "Jobs Pending: ${user_jobs_pending}"
    echo "Number of CPUs Running: ${user_cpus_running}"
    echo "Number of CPUs Pending: ${user_cpus_pending}"
    echo "-------------------"
}
# Prints specific partition info
function print_partition ()
{
    PARTITION="${1}"
    squeue -p ${PARTITION} -o "%15i %5C %5D %13P %10u %15j %.4t %.10M %.17R %10Q"

    available=`sinfo -h -o "%C" -p ${PARTITION} | awk -F "/" '{print $2}'`
    
    running_jobs=`squeue -o "%7P %7C %7t" | awk -v part=${PARTITION} '{if(($1 == part)&&($3 == "R")) print $0}' | wc -l`
    running_cpus=`squeue -o "%7P %7C %7t" | awk -v part=${PARTITION} '{if(($1 == part)&&($3 == "R")) print $2}' | awk 'BEGIN{count=0}{count=count+$1}END{print count}'`
    pending_jobs=`squeue -o "%7P %7C %7t" | awk -v part=${PARTITION} '{if(($1 == part)&&($3 == "PD")) print $0}' | wc -l`
    pending_cpus=`squeue -o "%7P %7C %7t" | awk -v part=${PARTITION} '{if(($1 == part)&&($3 == "PD")) print $2}' | awk 'BEGIN{count=0}{count=count+$1}END{print count}'`
    
    echo "-------------------"
    echo "PARTITION: ${PARTITION}"
    echo "Available CPUs: ${available}"
    echo "Running Jobs: ${running_jobs}"
    echo "Running CPUs: ${running_cpus}"
    echo "Pending Jobs: ${pending_jobs}"
    echo "Pending CPUs: ${pending_CPUs}"
    echo "-------------------"
    exit
}
# Prints info about current user
function print_default ()
{
    squeue -u ${USER} -o "%15i %5C %5D %10P %10u %15j %.4t %.10M %.17R %10Q"
    user_jobs_running=`squeue -h -u ${USER} -t R -o "%C" | wc -l`
    user_cpus_running=`squeue -h -u ${USER} -t R -o "%C" | awk 'BEGIN{count=0}{count=count+$1}END{print count}'`
    user_jobs_pending=`squeue -h -u ${USER} -t PD -o "%C" | wc -l`
    user_cpus_pending=`squeue -h -u ${USER} -t PD -o "%C" | awk 'BEGIN{count=0}{count=count+$1}END{print count}'`
    echo "-------------------"
    echo "USER: ${USER}"
    echo "Jobs Running: ${user_jobs_running}"
    echo "Jobs Pending: ${user_jobs_pending}"
    echo "Number of CPUs Running: ${user_cpus_running}"
    echo "Number of CPUs Pending: ${user_cpus_pending}"
    echo "-------------------"
}
#########################
##### END FUNCTIONS #####
#########################

# Parse options
PARSED_OPTIONS=$(getopt -n "$0" -o hu:sp: --long "help,user,summary,partition:,all,Total" -- "$@")
if [ "$?" -ne 0 ]; then
        exit # Exit if getopt failed
fi
eval set -- "$PARSED_OPTIONS"

# Call corresponding functions
while true; do
    case "$1" in
        -h | --help )
            echo ""
            echo "File location: $0"
            echo "Simple script to get queue system info from slurm"
            echo "-------------------"
            echo ""
            echo "Options:"
            echo ""
            echo "-h, -- help       : Prints this help message"
            echo ""
				echo "-u, --user        : Prints detailed info of user's jobs (needs to specify user)"
            echo ""
				echo "-s, --summary     : Summarizes usage of available partitions (needs no argument)"
            echo ""
				echo "-p, --partition   : Prints jobs and summary of a specific partition (needs to specify partition)"
            echo ""
            echo "If no option is specified, summary of user's usage is shown"
            echo ""
            exit
            shift;;
        -u | --user )
            print_user $2
            exit
            shift;;
        -s | --summary )
				print_summary
            exit
            shift;;
        -p | --partition )
            print_partition $2
            exit
            shift;;
        -- )
            print_default
            exit
            shift;;
esac
done
