# Network Device Sweep Automation &amp; Intrusion Detection Lab

This README walks you through a hands-on cybersecurity lab simulating a real-world scenario at SecureNet Corp, a mid-sized enterprise that recently faced a network intrusion. As a cybersecurity analyst, the task was to create a network sweep script that identifies active devices on a subnet, restrict its execution to security team members only, and log the results for comparison against the company's inventory list.

## Tools Used
- Kali Linux (Virtual Machine)

## Step-by-Step Process

### Part 1: Security Group Setup

#### 1: Created the Security Group
```bash
sudo groupadd security
```
Creates a new group called `security` to organize the security team members.


#### 2: Created Security Users
```bash
sudo useradd -m caro_sec
sudo useradd -m bernice_sec
sudo useradd -m irene_sec
```

#### 3: Added Users to the Security Group
```bash
sudo usermod -aG security caro_sec
sudo usermod -aG security bernice_sec
sudo usermod -aG security irene_sec
```
![sec group](https://github.com/user-attachments/assets/220d5d6f-dc2d-486f-8b5c-05435e8ea714)

### Part 2: Network Sweep Script

#### 1: Created the Script Directory and File(testsweep.sh)
```bash
sudo mkdir -p /home/shared/security
sudo touch /home/shared/security/testsweep.sh
```

#### 2: Wrote the Script
```bash
mousepad testsweep.sh
```

Opens a text editor where you write the script content and save when done

Script contents:

![sweep content](https://github.com/user-attachments/assets/2a7466f9-867e-4213-9e5a-b6a26c52970a)


**Breakdown of the script:**
- `#!/bin/bash` — tells the system this is a bash script
- `if [ -z "$1" ]` — checks if the user provided a subnet argument. If not, it reminds them of the correct syntax
- `$1` — represents the subnet the user passes when running the script (e.g. `10.0.2`), making the script flexible for any network
- `seq 1 254` — loops through every possible host on the subnet from `.1` to `.254`
- `grep "64 bytes"` — filters only successful responses
- `cut -d " " -f 4` — extracts the IP address from the ping output
- `tr -d ":"` — removes the colon from the extracted IP
- `&` — runs all pings in parallel for speed
- `done` — ends the loop
- `sort` — sorts the discovered IP addresses
- `tee sweep_results.txt` — saves discovered IP addresses to `sweep_results.txt` while also displaying them on screen

#### 3: Made the Script Executable
```bash
sudo chmod +x /home/shared/security/testsweep.sh
```

#### 4: Restricted Execution to Security Group Only
```bash
sudo chown :security /home/shared/security/testsweep.sh
sudo chmod 750 /home/shared/security/testsweep.sh
```

`chmod 750` means:

| Number | Who | Permissions |
|--------|-----|-------------|
| 7 | Owner | rwx |
| 5 | Security Group | r-x |
| 0 | Others | --- |

Only members of the `security` group can read and execute the script. Everyone else has no access.


#### 5: Run the Script
```bash
/home/shared/security/testsweep.sh 10.0.2
```

![testsweep](https://github.com/user-attachments/assets/beb64e9f-34e0-4563-9fb4-b6e49da96cdc)

**Note for you Dear curious mind:** Not sure what your subnet is? Open your terminal and run `ip a`, look for a line that starts with `inet` followed by numbers like `10.0.2.15`. The first three parts `(10.0.2)` is your subnet. Replace `10.0.2` with yours and you're good to go 😊

#### 6: Verified the Results
```bash
cat sweep_results.txt
```
![sec results](https://github.com/user-attachments/assets/b488590c-b2fa-4c4a-8a07-d2b32d4ecfd0)



## Key Observations / Lessons Learned

- **Bash scripting is powerful:** A few lines of code can automate a task that would take hours to do manually.
  
- **Arguments make scripts flexible:** Using `$1` instead of hardcoding a subnet means the script works for any network.

- **Running processes in parallel:** Adding `&` to run pings simultaneously instead of one at a time made a huge difference in speed. Without it, sweeping 254 addresses would take several minutes.

- **Access control is a security tool in itself:** Restricting who can execute a sensitive script like a network sweep is just as important as writing the script correctly. The wrong person running it could cause serious issues in a real environment(an observation that carries over from Lab 1😄).

- **Real-World Application:** This lab mirrors exactly what a cybersecurity analyst would do after a network intrusionL: sweep the network, identify active devices, and compare against an inventory list to find unauthorized devices.
