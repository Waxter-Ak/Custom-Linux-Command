#!/bin/bash

# Section A - Command Documentation
function display_help() {
  echo "internsctl v0.1.0 - Custom Linux Command"
  echo "Usage: internsctl [command] [options]"
  echo
  echo "Commands:"
  echo "  cpu getinfo               Display CPU information"
  echo "  memory getinfo            Display memory information"
  echo "  user create <username>    Create a new user"
  echo "  user list                 List all regular users"
  echo "  user list --sudo-only     List users with sudo permissions"
  echo "  file getinfo <file-name>  Display information about a file"
  echo "Options:"
  echo "  --size, -s                Print file size"
  echo "  --permissions, -p         Print file permissions"
  echo "  --owner, -o               Print file owner"
  echo "  --last-modified, -m       Print last modified time"
  echo
}

# Section B - Command Execution
function execute_command() {
  case $1 in
    "cpu" )
      lscpu
      ;;
    "memory" )
      free
      ;;
    "user" )
      case $2 in
        "create" )
          if [ -z "$3" ]; then
            echo "Error: Please provide a username."
            exit 1
          fi
          sudo useradd -m "$3"
          echo "User '$3' created successfully."
          ;;
        "list" )
          if [ "$3" == "--sudo-only" ]; then
            getent group sudo | cut -d: -f4 | tr ',' '\n'
          else
            getent passwd | cut -d: -f1
          fi
          ;;
        * )
          echo "Error: Invalid user command."
          exit 1
          ;;
      esac
      ;;
    "file" )
      if [ -z "$3" ]; then
        echo "Error: Please provide a file name."
        exit 1
      fi
      file_info=""
      for option in "${@:4}"; do
        case $option in
          "--size" | "-s" )
            file_info+="Size(B): $(du -b "$3" | cut -f1)"
            ;;
          "--permissions" | "-p" )
            file_info+="Access: $(stat -c %A "$3")"
            ;;
          "--owner" | "-o" )
            file_info+="Owner: $(stat -c %U "$3")"
            ;;
          "--last-modified" | "-m" )
            file_info+="Modify: $(stat -c %y "$3")"
            ;;
          * )
            echo "Error: Invalid option '$option' for file getinfo."
            exit 1
            ;;
        esac
      done
      if [ -z "$file_info" ]; then
        echo "File: $3"
        stat "$3"
      else
        echo "File: $3"
        echo "$file_info"
      fi
      ;;
    * )
      echo "Error: Invalid command."
      exit 1
      ;;
  esac
}

# Section A - Handle Command-line Arguments
case $1 in
  "--help" )
    display_help
    ;;
  "--version" )
    echo "internsctl v0.1.0"
    ;;
  * )
    execute_command "$@"
    ;;
esac
