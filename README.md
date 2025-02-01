#!/bin/bash

# Set thresholds
CPU_THRESHOLD=80  
MEM_THRESHOLD=80  

# Set number of iterations (e.g., 5 checks)
MAX_ITERATIONS=5
count=0

echo "Monitoring system performance..."
while (( count < MAX_ITERATIONS )); do
    # Get CPU usage
    CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}')
    
    # Get Memory usage
    MEM_USAGE=$(free | awk '/Mem/ {printf "%.2f", $3/$2 * 100}')

    # Check CPU usage against threshold
    if (( $(echo "$CPU_USAGE > $CPU_THRESHOLD" | bc -l) )); then
        echo "⚠️ High CPU Usage Detected: $CPU_USAGE%"
        echo "Top CPU Processes:"
        ps -eo pid,comm,%cpu --sort=-%cpu | head -5
    else
        echo "✅ System is running within normal limits. CPU Usage: $CPU_USAGE%"
    fi

    # Check Memory usage against threshold
    if (( $(echo "$MEM_USAGE > $MEM_THRESHOLD" | bc -l) )); then
        echo "⚠️ High Memory Usage Detected: $MEM_USAGE%"
        echo "Top Memory Processes:"
        ps -eo pid,comm,%mem --sort=-%mem | head -5
    else
        echo "✅ System is running within normal limits. Memory Usage: $MEM_USAGE%"
    fi

    # Increment count and wait 5 seconds
    count=$((count + 1))
    sleep 5
done

echo "Performance monitoring finished after $MAX_ITERATIONS checks."
