# 1. Project: Code Compilation Program

- Description: A set of automated processes that streamline the process of code review, compiling, building, packaging, and transfering to deployment environments.

- Code Snippet (Groovy):
The piece of code manages a set of parameters related to available compile nodes that are read from the parameter configuration file. It performs a condition check and constructs a list of allowed nodes based on the specific condition. Then these allowed nodes can be displayed on the Jenkins pipeline build's parameter page.

```java
GroovyShell shell = new GroovyShell()
def params = shell.parse(new File("..."))
compile_nodes = params.compile_nodes_params()
// 
compile_nodes_names = compile_nodes.keySet() as ArrayList
allowed_nodes = []
for (i in compile_nodes_names) {
    if (i.contains("ps3")) {
        allowed_nodes.add(i + ":disabled")
    } else {
        allowed_nodes.add(i)
    }
}
```

- Code Snippet (Bash):
This piece of code checks for existence of a flag file, which acts as a lock that controls access to the network gateway.

```bash
ls $flag_dir/flag_*

# Check if the flag file exists
if [[ $? == 0 ]]
then
	# Retrieve a list of filenames removing the "flag_" prefix
	# i.e. process IDs
	flags=($(ls $flag_dir/flag_* | cut -c31-))
	for i in ${flags[@]}
	do
		ps -p $i
		if [[ $? != 0 ]]
		then 
			# Clean the corrsponding flag file if the process does not exist
			rm -f $flag_dir/flag_$i
			echo "Process $i does not exist, file flag_$i cleaned"
		fi
	done
fi
```

# 2. Project: Material Management and Machine Monitoring Web System

- Description: The material management system primarily aims to record and maintain information about various components of an IT infrastructure, enabling efficient tracking of the movement, usage, and status. The machine monitoring system collected and analyzed data related to the performance of machines and integrated Jenkins automated test results, providing insights into the health, stability, and efficiency of the hosts in the IT environment.

- Code Snippet (HTML):
This piece of code iterates through each entry in the "host_info" model from Django database and generates an HTML table.
```html
{% for entry in host_info %}
    <tr>
        <td style="white-space: nowrap;">{{ forloop.counter }}</td>
        <td style="white-space: nowrap;">{{ entry.host_type }}</td>
        <td style="white-space: nowrap;">{{ entry.serial_number }}</td>
        <td style="white-space: nowrap;">{{ entry.storage_location|default_if_none:"" }}</td>
        <td style="white-space: nowrap;">{{ entry.department|default_if_none:"" }}</td>
        <td style="white-space: nowrap;">{{ entry.person_in_charge|default_if_none:"" }}</td>
        <td style="white-space: nowrap;">{{ entry.status|default_if_none:"" }}</td>
    </tr>
{% endfor %}

```
- Code Snippet (JavaScript)
This function utilizes jQuery for making asynchronous POST requests to update machine data and perform certain actions based on the response.

```javaScript
function update_data() {
	// Get the machine's IP address from a template variable
    var machine_ip = "{{node_ip}}"
    $.post('', 
    	{machine_ip: machine_ip},
        function (data) {
        	// Display a confirmation dialog with the returned data
            if (confirm(data)) {
                $.post('',
                    {ip:machine_ip, data: data},
                    function (data) {
                        if (data == 0){
                            alert('Machine hardware has not changed.')
                        }
                        else {
                            alert('Machine hardware change complete. Please verify hardware details carefully.')
                        }
                    }
                )
            }
        }
    )
}
```

This code provides a flexible way to perform conditional operations based on operators =, and, and or. 
```javaScript
// Define a hash table mapping operators to corresponding functions
var operatorHashTable = {
    '=': function(a, b) {return a == b;},
    'and': function (a, b) {return a && b},
    'or': function (a, b) {return a || b}
}

function cond_operation(op, oprand1, oprand2) {
    if (oprand1 == null) {
    	// If operand1 is null, return operand2
        return oprand2
    } else if (oprand2 == null) {
    	// If operand2 is null, return operand1
        return oprand1
    } else if (oprand1 != null && oprand2 != null) {
    	// If both operands are not null, use the operatorHashTable to perform the operation
        return operatorHashTable[op](oprand1, oprand2)
    }
}
```
- Code Snippet (Python):
This function checks the synchronization status of a remote host's time with a specified chronyd time server.
```python
def check_chronyd_status(ip, chronyd_server):
	# Check the status of chronyd service on the remote host
    chronyd_cmd = f"ssh {ip} systemctl status chronyd.service | grep {chronyd_server}"
    chronyd_status, _, _ = shell_utils.execute_command_rtn(cmd=chronyd_cmd)

    # Check if the host's time synchronizes with the server
    if chronyd_status == 0:
        status = "SYNC"
    else:
        ping_cmd = f"ping -c 4 {ip}"
        ping_status, _, _ = shell_utils.execute_command_rtn(cmd=ping_cmd)
        
        # Check if the host is offline or asynchronous
        if ping_status == 0:
            status = "ASYNC"
        else:
            status = "OFFLINE"
    return status
```

# 3. Project: Web-based Automation Testing Tool
- Description: The web-based automation testing tool primarily aims to offer a graphical interface for testing the functionality of storage-related components, including physical drives, virtual drives, and RAID.

- Code Snippet (Python, Selenium):
It seems to be designed to control the number of items displayed on a web page using a dropdown menu.
```python
def display_items_on_page(self, num):
    if num not in [10, 20, 50, 100]:
        raise Exception("Invalid number of items")
    # Find page_menus elements using the given XPath
    page_menus = self.find_elements_by_xpath(
        '//div[@class="ofs-select ofs-select-sm ofs-pagination-options-size-changer ofs-select-single ofs-select-show-arrow"]')
    for i, e in enumerate(page_menus):
        # Check if the element is visible to users, to operate only on the top-level visible page
        if e.is_displayed():
            e.click()
            # Find and click the option that matches the desired number of items per page
            e.find_element_by_xpath(
                '//div[@class="ofs-select-item-option-content" and contains(text(), "{} / page")]'.format(
                    num)).click()
```

# 4. Project: Staff Management Web Application
- Description: The staff management web application aims to manage active directory accounts for development personnel, which effectively supports the MIS concept by optimizing IT resource management, enhancing security, and improving the overall efficiency of development operations.

- Code Snippet (Python):
This function changes user's group in active directory.

```python
def changeUserGroup(userid, fullname, groupid):
    # Retrieve the current group memberships of the specified user using the provided userid and universal domain name
    # AD_DN (global variable) is a distinguished name fragment that represents a specific location within an AD domain hierarchy
    cmd = 'dsget user "CN=%s,%s" -memberof' % (userid, AD_DN)
    try:
        output = subprocess.check_output(cmd, shell=True, universal_newlines=True)
    except Exception as e:
        # If the command execution encounters an exception, log the exception using the logging module and exit
        logging.exception(e)
        exit()
    # Extract the group names from the shell output using regular expression
    joinedGroups = re.findall(r"CN=(.+?),CN=", output)
    # Find the intersection of the user's current groups (joinedGroups) and a list of departments(a global variable)
    joinedDepart = list(set(joinedGroups).intersection(set(departments)))
    if len(joinedDepart) == 1:
        # If the user joins exactly one group and needs change group
        if not joinedDepart[0] == groupid:
            # Remove the user from current group, add them to the desired group, and log this action
            removeFromGroup(userid, joinedDepart[0]) 
            addToGroup(userid, groupid)
            logging.info("%s: %s ---> %s" % (fullname, joinedDepart[0], groupid))
    # If the user is not a member of any groups, log an error
    elif len(joinedDepart) == 0:
        logging.error("User %s has no department group." % (fullname))
        exit()
    # If the user is a member of more than one group, log an error
    else:
        logging.error("User %s exists in more than one department groups." % (fullname))
        exit()
```

# 5. Project: Ping Monitor Program
- Description: This program aims to ping a destination IP address from user input and record the latency using the InfluxDB database. The result is visualized on a graphical timeline within InfluxDB.

- Code Snippet (Python): 
```python
# Validate the format of an IP address using regular expression
pattern = re.compile("^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}$")
if ( bool(re.match(pattern, dest_ip)) == False ):
    print("Error: Invalid ip format.")
    exit()
while True:
    # Continuously ping the destination ip
    cmd = 'ping -c 1 "%s"' % (dest_ip) 
    process = subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
    output, _unused_err = process.communicate()
    retcode = process.poll()
    if ( retcode == 0):
        # Creat the data point with a latency value using the InfluxDB Python client library
        value = output.decode().strip().split('/')[-3]
        p = influxdb_client.Point("%s-%s" % (my_ip, dest_ip)).field("latency", float((value))) 
        write_api.write(bucket=bucket, org=org, record=p)
    else:
        # Creat the data point with a latency value of -1.0 to indicate failure
        p = influxdb_client.Point("%s-%s" % (my_ip, dest_ip)).field("latency", -1.0) 
        write_api.write(bucket=bucket, org=org, record=p)
        # Log an error message indicating the destination IP address is unreachable
        logging.error("Failed: %s is unreachable" % (dest_ip))
    # Pause the script for one second before the next iteration
    time.sleep(1)
```

