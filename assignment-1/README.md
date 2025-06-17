### 🚀 **Assignment 1: Automated Instance Management Using AWS Lambda and Boto3**

---

#### 🖥️ **Step 1: Launch and Tag EC2 Instances**

##### 1.1 🔐 Log in to AWS Console

Go to [AWS Console](https://console.aws.amazon.com/).

##### 1.2 💡 Launch Two EC2 Instances

1. **Navigate:** 🔎 Search for and select **EC2** from the AWS services.
2. **Launch Instance:** 🟢 Click **Instances** in the left sidebar, then click the **Launch Instances** button.
3. **Instance details:**

   * 🏷️ Name: `AutoStopInstance` (for first instance)
   * 💻 AMI: Choose Amazon Linux 2 (or any free-tier OS)
   * 🪙 Instance type: `t2.micro` (free tier)
   * 🗝️ Key pair: Select/Create as needed
   * 🌐 Network settings: Default is fine for demo
4. **Add tag for first instance:**

   * Scroll down to the **Tags** section.
   * ➕ Click **Add tag**.

     * **Key:** `Action`
     *  **Value:** `Auto-Stop`
   * ➕ Click **Add tag**.

     * **Key:** `USER`
     *  **Value:** `Jasmine`
5. **Repeat** to launch a second instance:

   * 🏷️ Name: `AutoStartInstance`
   * **Tag:**

     * **Key:** `Action`
     * **Value:** `Auto-Start`
   * ➕ Click **Add tag**.

     * **Key:** `USER`
     *  **Value:** `Jasmine`
6. 🚀 **Launch Instances** and wait for both to be in **running** state.
7. ![launch Auto-stop](https://github.com/user-attachments/assets/9a3fc414-4bd7-4680-bb86-bcf5b8f36756)

8. ![launchAuto-start](https://github.com/user-attachments/assets/35b76dcb-c6f2-42eb-afd4-b85bb973e4d8)


---

#### 🔐 **Step 2: Create IAM Role for Lambda**

##### 2.1 🧑‍💻 Go to IAM

1. In AWS Console, 🔎 search and select **IAM**.
2. Click **Roles** > **Create Role**.
![creating role](https://github.com/user-attachments/assets/520cfc0d-b03b-49c2-8efc-b40b22f68fc5)
##### 2.2 🛡️ Create Role

1. **Trusted entity:** Choose **AWS service**
2. **Use Case:** Select **Lambda**.
3. **Next: Permissions**
![Create Role](images/role-2.png)
##### 2.3 📜 Attach Permissions

1. **Policy:** Search for and select `AmazonEC2FullAccess`.
   *(For production: use custom least-privilege policy.)*
2. Click **Next**
3. 📝 **Role Name:** Example: `JasmineLambdaEC2ManagementRole`
4. ✅ Click **Create Role**
![attachpolicy   role creation](https://github.com/user-attachments/assets/f3434913-ad2f-431f-b990-d9c9ca60de82)

---

#### ⚡ **Step 3: Create Lambda Function**

##### 3.1 🏃‍♂️ Go to Lambda Console

1. In AWS Console, 🔎 search for and select **Lambda**.
2. Click **Create function**

##### 3.2 ⚙️ Configure Function

1. **Author from scratch**

   * 📝 Name: `EC2AutoStartStop`
   * 🐍 Runtime: **Python 3.12**
2. **Change default execution role:**

   * Select **Use an existing role**
   * Choose the `JasmineLambdaEC2ManagementRole` you just created
3. ✅ Click **Create function**
![Function creation](https://github.com/user-attachments/assets/4697cfce-5fd2-4882-b4d4-319197fa6a70)

---

#### 💻 **Step 4: Add Lambda Code**

##### 4.1 ✏️ Open Function Editor

* Scroll down to the **Code source** section.

##### 4.2 📝 Paste the Code

Delete any default code and paste the following:

```python
import boto3

def lambda_handler(event, context):
    # Create an EC2 client
    ec2 = boto3.client('ec2')

    # Find instances with tag Action=Auto-Stop
    stop_response = ec2.describe_instances(
        Filters=[
            {'Name': 'tag:Action', 'Values': ['Auto-Stop']},
            {'Name': 'tag:USER', 'Values': ['Jasmine']},
            {'Name': 'instance-state-name', 'Values': ['running']}
        ]
    )

    instances_to_stop = [
        instance['InstanceId']
        for reservation in stop_response['Reservations']
        for instance in reservation['Instances']
    ]

    # Find instances with tag Action=Auto-Start
    start_response = ec2.describe_instances(
        Filters=[
            {'Name': 'tag:Action', 'Values': ['Auto-Start']},
            {'Name': 'tag:USER', 'Values': ['Jasmine']},
            {'Name': 'instance-state-name', 'Values': ['stopped']}
        ]
    )

    instances_to_start = [
        instance['InstanceId']
        for reservation in start_response['Reservations']
        for instance in reservation['Instances']
    ]

    # Stop instances
    if instances_to_stop:
        print(f"Stopping: {instances_to_stop}")
        ec2.stop_instances(InstanceIds=instances_to_stop)
    else:
        print("No instances to stop.")

    # Start instances
    if instances_to_start:
        print(f"Starting: {instances_to_start}")
        ec2.start_instances(InstanceIds=instances_to_start)
    else:
        print("No instances to start.")

    # Print the results
    return {
        'StoppedInstances': instances_to_stop,
        'StartedInstances': instances_to_start
    }
```
![Deploying code](https://github.com/user-attachments/assets/d0beb9d8-3d67-4683-88e1-9ffc159843c2)

##### 4.3 🚀 **Click Deploy**

---

#### 🧪 **Step 5: Test Your Lambda Function**

##### 5.1 🟢 Prepare Test State

* Ensure your "Auto-Stop" instance is **running** and "Auto-Start" is **stopped** for visible effect.
!![success logs of function](https://github.com/user-attachments/assets/b75b7bab-e067-48ce-ac3b-1a9af359d392)

##### 5.2 🧑‍🔬 Test in Lambda Console

1. In your Lambda function page, click **Test**.
2. For the first time, it asks to "Configure test event":

   * 📝 **Event name:** (give any name)
   * Leave the event JSON as `{}` (empty event)
   * Click **Save**
3. 🟢 Click **Test** (again) to **run** the function.
![creationof testevent](https://github.com/user-attachments/assets/30864d22-b67d-4724-90ae-e495ae3f77e9)

##### 5.3 🔍 Check the Output

* Scroll down to **Execution results**: You should see the list of instance IDs started/stopped.
* Go to **EC2 dashboard** and confirm:

  * Instance with tag `Auto-Stop` is **stopping**
  * Instance with tag `Auto-Start` is **starting**
![Autostartrunning](https://github.com/user-attachments/assets/c8dd0a93-2812-4b1d-9491-6929eeb3a1af)

##### 5.4 📄 View Logs (Optional)

* Go to **Monitor > View logs in CloudWatch** to see Lambda’s print output.
