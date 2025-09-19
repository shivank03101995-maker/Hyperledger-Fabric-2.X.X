# **🔹 Install Hyperledger Fabric Binaries**

**Step 1: Make a workspace**
    
    cd ~
    mkdir fabric-samples && cd fabric-samples

**Step 2: Download binaries**

    Run the official installation script:

    curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.5.12 1.5.15


This will install:

    peer
    orderer
    configtxgen
    cryptogen
    fabric-ca-client
    fabric-ca-server
    And place them in:
    ~/fabric-samples/bin/

**Step 3: Add binaries to PATH**

To use Fabric CLI commands globally, add this to your shell profile (~/.bashrc or ~/.zshrc):

    export PATH=$PATH:~/fabric-samples/bin


Then reload your profile:

    source ~/.bashrc

**Step 4: Verify installation**

Check the installed versions:

    peer version
    orderer version
    configtxgen --version
    cryptogen version
    fabric-ca-client version


✅ At this point, you have only the binaries installed.
You can now use them later to generate crypto material, create channels, or run a network manually.