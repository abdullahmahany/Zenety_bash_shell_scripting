#!/bin/bash

# Main Menu using Zenity
function main_menu {
    choice=$(zenity --list \
        --title="Main Menu" \
        --text="Choose an option:" \
        --column="Option" \
        "Create Database" \
        "List Databases" \
        "Connect To Database" \
        "Drop Database" \
        "Exit")

    case $choice in
        "Create Database") create_database ;;
        "List Databases") list_databases ;;
        "Connect To Database") connect_database ;;
        "Drop Database") drop_database ;;
        "Exit") exit ;;
        *) zenity --error --text="Invalid choice. Please try again." ;;
    esac
}

# Create Database
function create_database {
    dbname=$(zenity --entry --title="Create Database" --text="Enter database name:")
    if [[ -z "$dbname" ]]; then
        zenity --error --text="Database name cannot be empty."
    else
        mkdir -p "databases/$dbname"
        zenity --info --text="Database '$dbname' created."
    fi
    main_menu
}

# List Databases
function list_databases {
    dblist=$(ls -1 "databases" 2>/dev/null)
    if [[ -z "$dblist" ]]; then
        zenity --info --text="No databases found."
    else
        zenity --list --title="Databases" --text="List of databases:" --column="Databases" $dblist
    fi
    main_menu
}

# Connect To Database
function connect_database {
    dbname=$(zenity --entry --title="Connect To Database" --text="Enter database name:")
    if [[ -d "databases/$dbname" ]]; then
        database_menu "$dbname"
    else
        zenity --error --text="Database '$dbname' does not exist."
        main_menu
    fi
}

# Drop Database
function drop_database {
    dbname=$(zenity --entry --title="Drop Database" --text="Enter database name:")
    if [[ -d "databases/$dbname" ]]; then
        rm -r "databases/$dbname"
        zenity --info --text="Database '$dbname' dropped."
    else
        zenity --error --text="Database '$dbname' does not exist."
    fi
    main_menu
}

# Database Menu using Zenity
function database_menu {
    dbname=$1
    choice=$(zenity --list \
        --title="Database Menu for '$dbname'" \
        --text="Choose an option:" \
        --column="Option" \
        "Create Table" \
        "List Tables" \
        "Drop Table" \
        "Insert into Table" \
        "Select From Table" \
        "Delete From Table" \
        "Update Table" \
        "Back to Main Menu")

    case $choice in
        "Create Table") create_table "$dbname" ;;
        "List Tables") list_tables "$dbname" ;;
        "Drop Table") drop_table "$dbname" ;;
        "Insert into Table") insert_into_table "$dbname" ;;
        "Select From Table") select_from_table "$dbname" ;;
        "Delete From Table") delete_from_table "$dbname" ;;
        "Update Table") update_table "$dbname" ;;
        "Back to Main Menu") main_menu ;;
        *) zenity --error --text="Invalid choice. Please try again." ;;
    esac
}

# Create Table
function create_table {
    dbname=$1
    tablename=$(zenity --entry --title="Create Table" --text="Enter table name:")
    if [[ -z "$tablename" ]]; then
        zenity --error --text="Table name cannot be empty."
    else
        columns=$(zenity --entry --title="Create Table" --text="Enter columns (e.g., id:int:pk,name:str,age:int):")
        if [[ -z "$columns" ]]; then
            zenity --error --text="Columns cannot be empty."
        else
            # Check if a primary key is specified
            if [[ ! "$columns" =~ :pk ]]; then
                zenity --error --text="No primary key specified. Please mark one column as 'pk' (e.g., id:int:pk)."
            else
                echo "$columns" > "databases/$dbname/$tablename"
                zenity --info --text="Table '$tablename' created."
            fi
        fi
    fi
    database_menu "$dbname"
}

# List Tables
function list_tables {
    dbname=$1
    tablelist=$(ls -1 "databases/$dbname" 2>/dev/null)
    if [[ -z "$tablelist" ]]; then
        zenity --info --text="No tables found in database '$dbname'."
    else
        zenity --list --title="Tables in '$dbname'" --text="List of tables:" --column="Tables" $tablelist
    fi
    database_menu "$dbname"
}

# Drop Table
function drop_table {
    dbname=$1
    tablename=$(zenity --entry --title="Drop Table" --text="Enter table name:")
    if [[ -f "databases/$dbname/$tablename" ]]; then
        rm "databases/$dbname/$tablename"
        zenity --info --text="Table '$tablename' dropped."
    else
        zenity --error --text="Table '$tablename' does not exist."
    fi
    database_menu "$dbname"
}

# Insert into Table
function insert_into_table {
    dbname=$1
    tablename=$(zenity --entry --title="Insert into Table" --text="Enter table name:")
    if [[ -f "databases/$dbname/$tablename" ]]; then
        columns=$(cat "databases/$dbname/$tablename")
        IFS=',' read -r -a cols <<< "$columns"
        values=""
        pk_col_index=-1

        # Find the primary key column index
        for i in "${!cols[@]}"; do
            if [[ "${cols[$i]}" =~ :pk$ ]]; then
                pk_col_index=$i
                break
            fi
        done

        if [[ $pk_col_index -eq -1 ]]; then
            zenity --error --text="No primary key column found in the table."
            database_menu "$dbname"
            return
        fi

        # Collect values for each column
        for i in "${!cols[@]}"; do
            IFS=':' read -r -a col_info <<< "${cols[$i]}"
            colname=${col_info[0]}
            coltype=${col_info[1]}
            is_pk=${col_info[2]} # Check if this column is the primary key

            while true; do
                value=$(zenity --entry --title="Insert into Table" --text="Enter value for '$colname' ($coltype):")
                if [[ -z "$value" && "$is_pk" == "pk" ]]; then
                    zenity --error --text="Primary key cannot be empty. Please enter a valid value."
                elif [[ "$coltype" == "int" && ! "$value" =~ ^[0-9]+$ ]]; then
                    zenity --error --text="Invalid input. '$colname' must be an integer."
                else
                    # Check if the primary key is unique
                    if [[ "$is_pk" == "pk" ]]; then
                        pk_value="$value"
                        if grep -q "^$pk_value," "databases/$dbname/$tablename"; then
                            zenity --error --text="Primary key '$pk_value' already exists. Please enter a unique value."
                        else
                            break
                        fi
                    else
                        break
                    fi
                fi
            done
            values+="$value,"
        done
        values=${values%,}
        echo "$values" >> "databases/$dbname/$tablename"
        zenity --info --text="Record inserted."
    else
        zenity --error --text="Table '$tablename' does not exist."
    fi
    database_menu "$dbname"
}

# Select From Table
function select_from_table {
    dbname=$1
    tablename=$(zenity --entry --title="Select From Table" --text="Enter table name:")
    if [[ -f "databases/$dbname/$tablename" ]]; then
        zenity --text-info --title="Records in '$tablename'" --filename="databases/$dbname/$tablename"
    else
        zenity --error --text="Table '$tablename' does not exist."
    fi
    database_menu "$dbname"
}

# Delete From Table
function delete_from_table {
    dbname=$1
    tablename=$(zenity --entry --title="Delete From Table" --text="Enter table name:")
    if [[ -f "databases/$dbname/$tablename" ]]; then
        columns=$(cat "databases/$dbname/$tablename")
        IFS=',' read -r -a cols <<< "$columns"
        pk_col_index=-1

        # Find the primary key column index
        for i in "${!cols[@]}"; do
            if [[ "${cols[$i]}" =~ :pk$ ]]; then
                pk_col_index=$i
                break
            fi
        done

        if [[ $pk_col_index -eq -1 ]]; then
            zenity --error --text="No primary key column found in the table."
            database_menu "$dbname"
            return
        fi

        pk=$(zenity --entry --title="Delete From Table" --text="Enter the primary key value to delete:")
        if [[ -z "$pk" ]]; then
            zenity --error --text="Primary key cannot be empty."
        else
            sed -i "/^$pk,/d" "databases/$dbname/$tablename"
            zenity --info --text="Record deleted."
        fi
    else
        zenity --error --text="Table '$tablename' does not exist."
    fi
    database_menu "$dbname"
}

# Update Table
function update_table {
    dbname=$1
    tablename=$(zenity --entry --title="Update Table" --text="Enter table name:")
    if [[ -f "databases/$dbname/$tablename" ]]; then
        columns=$(cat "databases/$dbname/$tablename")
        IFS=',' read -r -a cols <<< "$columns"
        pk_col_index=-1

        # Find the primary key column index
        for i in "${!cols[@]}"; do
            if [[ "${cols[$i]}" =~ :pk$ ]]; then
                pk_col_index=$i
                break
            fi
        done

        if [[ $pk_col_index -eq -1 ]]; then
            zenity --error --text="No primary key column found in the table."
            database_menu "$dbname"
            return
        fi

        pk=$(zenity --entry --title="Update Table" --text="Enter the primary key value to update:")
        if [[ -z "$pk" ]]; then
            zenity --error --text="Primary key cannot be empty."
        else
            record=$(grep "^$pk," "databases/$dbname/$tablename")
            if [[ -z "$record" ]]; then
                zenity --error --text="Record not found."
            else
                IFS=',' read -r -a values <<< "$record"
                new_values=""
                for i in "${!cols[@]}"; do
                    IFS=':' read -r -a col_info <<< "${cols[$i]}"
                    colname=${col_info[0]}
                    coltype=${col_info[1]}
                    is_pk=${col_info[2]} # Check if this column is the primary key

                    while true; do
                        value=$(zenity --entry --title="Update Table" --text="Enter new value for '$colname' ($coltype) [current: ${values[$i]}]:")
                        if [[ -z "$value" && "$is_pk" == "pk" ]]; then
                            zenity --error --text="Primary key cannot be empty. Please enter a valid value."
                        elif [[ "$coltype" == "int" && ! "$value" =~ ^[0-9]+$ ]]; then
                            zenity --error --text="Invalid input. '$colname' must be an integer."
                        else
                            break
                        fi
                    done
                    new_values+="$value,"
                done
                new_values=${new_values%,}
                sed -i "s/^$pk,.*$/$new_values/" "databases/$dbname/$tablename"
                zenity --info --text="Record updated."
            fi
        fi
    else
        zenity --error --text="Table '$tablename' does not exist."
    fi
    database_menu "$dbname"
}

# Start the application
main_menu
