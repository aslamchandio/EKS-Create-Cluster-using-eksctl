# Install AWS, kubectl & eksctl CLI's

## Step-00: Introduction
- Install AWS CLI
- Install kubectl CLI
- Install eksctl CLI

## Step-01: Install AWS CLI
- Reference-2: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-version.html
### Step-01-01: Linux - Install and configure AWS CLI
- Download the binary and install via command line using below two commands. 
```
# Download Binary

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64-2.0.30.zip" -o "awscliv2.zip"

# Unzip the Packge 
unzip awscliv2.zip
...

# Install the binary
sudo ./aws/install
```
- Verify the installation 
```
# which aws
/usr/local/bin/aws
...

aws --version
aws-cli/2.10.0 Python/3.11.2 Linux/4.14.133-113.105.amzn2.x86_64 botocore/2.4.5

```

### Step-01-02: Configure AWS Command Line using Security Credentials
- Go to AWS Management Console --> Services --> IAM
- Select the IAM User: kalyan 
- **Important Note:** Use only IAM user to generate **Security Credentials**. Never ever use Root User. (Highly not recommended)
- Click on **Security credentials** tab
- Click on **Create access key**
- Copy Access ID and Secret access key
- Go to command line and provide the required details
```
aws configure
AWS Access Key ID [None]: ABCDEFGHIAZBERTUCNGG  (Replace your creds when prompted)
AWS Secret Access Key [None]: uMe7fumK1IdDB094q2sGFhM5Bqt3HQRw3IHZzBDTm  (Replace your creds when prompted)
Default region name [None]: us-east-1
Default output format [None]: json
```
- Test if AWS CLI is working after configuring the above
```
aws ec2 describe-vpcs
```

## Step-02: Install kubectl CLI
- **IMPORTANT NOTE:** Kubectl binaries for EKS please prefer to use from Amazon (**Amazon EKS-vended kubectl binary**)
- This will help us to get the exact Kubectl client version based on our EKS Cluster version. You can use the below documentation link to download the binary.
- Reference: https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

### Step-02-01: Linux - To install or update kubectl on Linux (amd64)
- Kubectl version we are using here is 1.28.3 or 1.27.7(It may vary based on Cluster version you are planning use in AWS EKS)

```
# Download the Package

curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.28.3/2023-11-14/bin/linux/amd64/kubectl

curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.7/2023-11-14/bin/linux/amd64/kubectl

- # (Optional) Verify the downloaded binary with the SHA-256 checksum for your binary.

curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.28.3/2023-11-14/bin/linux/amd64/kubectl.sha256

curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.7/2023-11-14/bin/linux/amd64/kubectl.sha256

- # Check the SHA-256 checksum for your downloaded binary with one of the following commands.

sha256sum -c kubectl.sha256

When using this command, make sure that you see the following output:

kubectl: OK


# Apply execute permissions to the binary.
chmod +x ./kubectl

# Copy the binary to a folder in your PATH. If you have already installed a version of kubectl, then we recommend creating a $HOME/bin/kubectl and ensuring that $HOME/bin comes first in your $PATH.

mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH

# (Optional) Add the $HOME/bin path to your shell initialization file so that it is configured when you open a shell.

echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc


# After you install kubectl, you can verify its version.

kubectl version --client

### Step-02-02: Linux - To install or update kubectl on Linux (amd64) from Kubernetes.io

# Download the latest release with the command:

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

Note:
To download a specific version, replace the $(curl -L -s https://dl.k8s.io/release/stable.txt) portion of the command with the specific version.

For example, to download version 1.29.0 on Linux x86-64, type:

curl -LO https://dl.k8s.io/release/v1.29.0/bin/linux/amd64/kubectl

# Validate the binary (optional)

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

# Validate the kubectl binary against the checksum file:

echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check

# If valid, the output is:

kubectl: OK

# Install kubectl

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

Note:
If you do not have root access on the target system, you can still install kubectl to the ~/.local/bin directory:

chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl
# and then append (or prepend) ~/.local/bin to $PATH

# Test to ensure the version you installed is up-to-date:

kubectl version --client

Or use this for detailed view of version:

kubectl version --client --output=yaml


Reference-1: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/



```

### Step-02-02: Windows 10 - Install and configure kubectl


# Download the latest 1.29 patch release: kubectl 1.29.0.

Or if you have curl installed, use this command:

curl.exe -LO "https://dl.k8s.io/release/v1.29.0/bin/windows/amd64/kubectl.exe"

Note: To find out the latest stable version (for example, for scripting), take a look at https://dl.k8s.io/release/stable.txt.

# Validate the binary (optional)

Download the kubectl checksum file:

curl.exe -LO "https://dl.k8s.io/v1.29.0/bin/windows/amd64/kubectl.exe.sha256"

# Using Command Prompt to manually compare CertUtil's output to the checksum file downloaded:

CertUtil -hashfile kubectl.exe SHA256
type kubectl.exe.sha256

Using PowerShell to automate the verification using the -eq operator to get a True or False result:

$(Get-FileHash -Algorithm SHA256 .\kubectl.exe).Hash -eq $(Get-Content .\kubectl.exe.sha256)

# Test to ensure the version of kubectl is the same as downloaded:

kubectl version --client


## Step-03: Install eksctl CLI

...

### Step-03-01: eksctl on linux

To download the latest release, run:

# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo mv /tmp/eksctl /usr/local/bin

### Step-03-02: eksctl on Windows

Direct download (latest release): AMD64/x86_64 - ARMv6 - ARMv7 - ARM64¶

Make sure to unzip the archive to a folder in the PATH variable.

Optionally, verify the checksum:

1. Download the checksum file: latest
2. Use Command Prompt to manually compare CertUtil's output to the checksum file downloaded.

REM Replace amd64 with armv6, armv7 or arm64
CertUtil -hashfile eksctl_Windows_amd64.zip SHA256


3. Using PowerShell to automate the verification using the -eq operator to get a True or False result:

# Replace amd64 with armv6, armv7 or arm64
 (Get-FileHash -Algorithm SHA256 .\eksctl_Windows_amd64.zip).Hash -eq ((Get-Content .\eksctl_checksums.txt) -match 'eksctl_Windows_amd64.zip' -split ' ')[0]
 ```

#### Using Git Bash: 
```sh
# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=amd64
PLATFORM=windows_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.zip"

# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

unzip eksctl_$PLATFORM.zip -d $HOME/bin

rm eksctl_$PLATFORM.zip


## References:
- https://eksctl.io/getting-started/
- https://eksctl.io/installation/