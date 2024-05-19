# Script to terminate running tasks on a port
## Prerequisites
- `sudo` privileges for executing commands as root

## Setup
1. **Save the script**: Save the provided script to a file, for example, `terminate_process_on_port.sh`.

    ```bash
    #!/usr/bin/env bash

    # Prompt for sudo privileges at the beginning of the script
    if [ "$EUID" -ne 0 ]; then
        sudo "$0" "$@"
        exit $?
    fi

    # Check if port number is provided as argument
    if [ $# -ne 1 ]; then
        echo "Usage: $0 <port>"
        exit 1
    fi

    port=$1

    # Check if any process is running on the specified port
    if lsof -i :$port >/dev/null 2>&1; then
        # Get the PID(s) of the process(es) running on the port
        pids=$(lsof -t -i :$port)

        # Attempt to terminate the processes running on the specified port
        terminate_task() {
            local pid=$1
            local exec_name=$(ps -p $pid -o comm=)
            local process_name=$(ps -p $pid -o args=)
            
            kill $pid
            
            # Check if the process is still running after sending SIGTERM
            sleep 3
            if ps -p $pid >/dev/null; then
                kill -9 $pid
                echo "Process ($pid) - $exec_name ($process_name) did not terminate after SIGTERM and was forcefully killed with SIGKILL."
            else
                echo "Process ($pid) - $exec_name ($process_name) terminated successfully."
            fi
        }

        for pid in $pids; do
            terminate_task $pid
        done
    else
        echo "No processes found running on port $port."
    fi
    ```

2. **Set execute permissions**: Make the script executable using the command:
    ```bash
    chmod +x terminate_process_on_port.sh
    ```
### Optionals

- **Add alias**

    Be sure to change the `/path/to/terminate_process_on_port.sh` with actual path.

    - **For Bash**
      ```bash
      echo "alias freeport='/path/to/terminate_process_on_port.sh'" >> ~/.bashrc
      ```
      ```bash
      source ~/.bashrc
      ```
    - **For Zsh**
      ```bash
      echo "alias freeport='/path/to/terminate_process_on_port.sh'" >> ~/.zshrc
      ```
      ```bash
      source ~/.zshrc
      ```
    - **For Fish**
      ```bash
      echo "alias freeport '/path/to/terminate_process_on_port.sh'" >> ~/.config/fish/config.fish
      ```
      ```bash
      source ~/.config/fish/config.fish
      ```
### Or
- **Add to path**
    
    Move the script to local bin then add that path to env PATH.
    ```bash
    mkdir -p ~/.local/bin
    mv ~/terminate_process_on_port.sh ~/.local/bin/freeport
    ```

    - **For Bash**
      ```bash
      echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
      ```
      ```bash
      source ~/.bashrc
      ```
    - **For Zsh**
      ```bash
      echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
      ```
      ```bash
      source ~/.zshrc
      ```
    - **For Fish**
      ```bash
      echo 'set -Ux fish_user_paths $HOME/.local/bin $fish_user_paths' >> ~/.config/fish/config.fish
      ```
      ```bash
      source ~/.config/fish/config.fish
      ```

## Usage
Run the script with the following command:
```bash
./terminate_process_on_port.sh <port>
```
Replace `<port>` with the port number where you want to terminate the process running on.
>
>Run the following command if followed through any of the optional setup:
>```bash
>freeport <port>
>```

### Example:
To terminate a process running on port `8080`, run:
```bash
./terminate_process_on_port.sh 8080
```
>or
>```bash
>freeport 8080
>```