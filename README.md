
# SmartQuora
## Table of Contents
- [SmartQuora](#smartquora)
  * [Table of Contents](#table-of-contents)
  * [Introduction](#introduction)
  * [Audience](#audience)
  * [Architecture](#architecture)
    + [Storage](#storage)
    + [Service Layer](#service-layer)
    + [Authentication](#authentication)
    + [Digital Identity & Wallets](#digital-identity---wallets)
    + [Nodes](#nodes)
    + [Single and Multi Deployment View of the SmartQuora DApp](#single-and-multi-deployment-view-of-the-smartquora-dapp)
  * [DApps - Decentralized Applications](#dapps---decentralized-applications)
    + [Understanding Blockchain](#understanding-blockchain)
    + [Anatomy of the DApp (or BNA)](#anatomy-of-the-dapp--or-bna-)
    + [Hyperledger Composer Components](#hyperledger-composer-components)
  * [SmartQuora Domain Model](#smartquora-domain-model)
    + [SmartQuora Participants](#smartquora-participants)
    + [SmartQuora Assets](#smartquora-assets)
    + [SmartQuora Transactions](#smartquora-transactions)
    + [SmartQuora Events](#smartquora-events)
  * [Setup Instructions](#setup-instructions)
    + [Installing Hyperledger Fabric on AWS](#installing-hyperledger-fabric-on-aws)
      - [Launch an EC2 Instance](#launch-an-ec2-instance)
      - [Associate an Elastic IP](#associate-an-elastic-ip)
      - [Create Hyperuser](#create-hyperuser)
      - [Install the Hyperledger Fabric Pre-requisites](#install-the-hyperledger-fabric-pre-requisites)
      - [Install the Hyperledger Development Environment](#install-the-hyperledger-development-environment)
    + [Starting and Stopping Hyperledger Fabric](#starting-and-stopping-hyperledger-fabric)
  * [Creating a Hello World on Hyperledger Fabric](#creating-a-hello-world-on-hyperledger-fabric)
  * [Installing SmartQuora](#installing-smartquora)
    + [Securing the REST Server](#securing-the-rest-server)
      - [Google+ Authentication Configuration & Setup](#google--authentication-configuration---setup)
      - [Setting up the Credentials & Wallet Data Store using MongoDB](#setting-up-the-credentials---wallet-data-store-using-mongodb)
    + [Start the REST server](#start-the-rest-server)
    + [Installing the Web Server](#installing-the-web-server)
  * [Running SmartQuora](#running-smartquora)
    + [Signing into SmartQuora](#signing-into-smartquora)
    + [Creating Digital Identity Cards](#creating-digital-identity-cards)
    + [Adding the Digital Identity to your Wallet](#adding-the-digital-identity-to-your-wallet)
    + [Selecting a Digital Avatar](#selecting-a-digital-avatar)
    + [Ask a Question](#ask-a-question)
    + [Provide an Answer](#provide-an-answer)
    + [Vote for an Answer](#vote-for-an-answer)
    + [Award an Answer](#award-an-answer)
  * [Exercises](#exercises)
    + [Issue 1 - Dealing with _SpendThrifts_](#issue-1---dealing-with--spendthrifts-)
      - [Solution 1](#solution-1)
    + [Issue 2 - Dealing with _DoubleSpenders_](#issue-2---dealing-with--doublespenders-)
      - [Solution 2](#solution-2)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

## Introduction
**SmartQuora** is an application that enables knowledge sharing among participants while incentivizing answers that are meaningful and well-explained. *Inquirers* pose questions with a reward for the best answers and a due-date by which they are looking for an answer. *Responders* compete with each other to provide the best answers. Participants can like or dislike answers. When the due-date arrives the answers are tallied and the reward is shared proportionately among the responders such that the best answers gets the most earnings.  To avoid abuse of the platform, inquirers cannot answer their own questions and respondents cannot vote for their own answers. 

Technically speaking, SmartQuora is a *DApp* (Decentralized Application) built on top of the HLF - Hyperledger Fabric Blockchain decentralized peer-to-peer network. It uses Smart Contracts built using HLF Composer API to represent Questions and Answers which contains rules to manage the process and payout. 

SmartQuora uses a Javascript-based front-end web application to communicate withe the Blockchain platform on which the Smart Contracts reside using a RESTful interface. It uses Passport for authentication of participants using OAuth protocol and allows maintenance of their digital wallets through which the participants can manage their Digital Identities. These Digital Identities are generated and managed using the Hyperledger Fabric platform.

**Please note** that the intent behind this application was to demonstrate building Smart Contracts on the Hyperledger Fabric platform. The process documented here is not an exhaustive exercise in building a production quality application.   

## Audience
You are a Developer or Solutions Architect wanting to learn and build robust, secure and scaleable decentralized applications using open source framework such as Hyperledger Fabric to take full advantage of the autonomy that a blockchain protocol provides without a central point of failure.

As you are building this application, you will learn about the Hyperledger Fabric blockchain framework, Hyperledger Fabric Composer API, Passport authentication middleware, Docker containerization and various development scaffolding frameworks such as Yeoman, Loopback etc. discussed below.

## Architecture
The Hyperledger Fabric blockchain platform that SmartQuora uses for this demonstration is built on top of AWS EC2 platform. It uses Docker containers to host various parts of the Hyperledger Fabric components such as the endorser, committer, ledger  (store) , orderer (consensus service) and the chain-code.

The following diagram provides a high-level component diagram of the SmartQuora DApp. 

![SmartQuora Component Diagram](images/smartquora-components.png)

As such, the blockchain is a decentralized system consisting of many nodes that communicate with each other on a peer-to-peer network. The blockchain runs programs called chaincode, which holds state and ledger data, and executes transactions. The chaincode is the central element as transactions are operations invoked on the chaincode. Transactions have to be “endorsed” and only endorsed transactions may be committed and have an effect on the state. 

### Storage
DApps such as SmartQuora consists of Smart Contracts that are translated to chaincode. This chaincode is then deployed into the blockchain. Storage of chaincode and state information differs across blockchain platforms. The default mechanism of persistent state storage in Hyperledger Fabric is LevelDB or CouchDB. In addition to supporting chaincode operations to store and retrieve assets, CouchDB allows performing complex rich queries against the data stored in the blockchain. SmartQuora uses CouchDB.

### Service Layer
Aside from storage, a DApp requires a service layer to communicate with the chaincode on the blockchain and a front-end for user interaction. SmartQuora takes advantage of the Hyperledger Composer REST server which uses [Loopback](http://www.loopback.io) to generate a REST server and maps it to the transactions and queries on the DApp.

### Authentication
SmartQuora DApp uses the OAuth authentication strategy of the [Passport](http://www.passportjs.org/) authentication middleware to secure the REST server. Specifically, it uses the [Passport Google OAuth](https://github.com/jaredhanson/passport-google-oauth#readme) delegated authentication strategy allowing users to authenticate themselves using their Google account.

All information regarding authenticated users and their wallets is persisted in a [LoopBack](http://loopback.io/) data source by using a LoopBack connector. By default, the REST server uses the LoopBack "memory" connector to persist user information, which is lost when the REST server is terminated. To enable persistent storage of the authenticated users and their wallets, SmartQuora uses a MongoDB LoopBack connector that stores data in a highly available MongoDB data source online at [mLab](https://mlab.com/).  It should be noted that a local MongoDB cluster can be used instead of MLab as well. MLab was chosen to ease the implementation process.

### Digital Identity & Wallets
Blockchain uses Digital Identities to represent participants in the network. A identity is a digital certificate and private key. These identities are used to sign transactions on behalf of the participants on the blockchain network. Identities are assembled in an envelope called _business network cards_ along with the metadata and connection profile of the participant in Hyperledger Fabric. These business cards are then stored in wallets. As such, a participant can have multiple business cards in their wallet.

### Nodes 
The decentralized nature of a blockchain platform is what gives it the power to tolerate system failures, record transactions that cannot be altered retroactively without the alteration of subsequent blocks and the collusion of the network. Decentralization or peer-to-peer network requires multiple nodes to be added to the blockchain network. These nodes are the communication entities of the blockchain. There are three types of nodes in the Hyperledger Fabric blockchain:

1.  **Client**  : The client represents the entity that acts on behalf of an end-user. It orchestrates data and transactions between peers.
2.  **Peer**: A peer receives ordered state updates in the form of _blocks_ from the ordering service and maintain the state and the ledger. Peers can additionally take up a special role of an  **endorser** which uses endorsement policies to sign the transactions. Once the client receives enough signatures to satisfy the endorsement policy, it can submit the transaction (and the signatures) to be added to the ledger.
3.  **Orderer**: The orderer provides a shared _communication channel_ to clients and peers, offering a broadcast service for messages containing transactions and implements a delivery guarantee.

For further details on the basic workflow of a transaction inside a Hyperledger Fabric blockchain please refer to this [document](http://hyperledger-fabric.readthedocs.io/en/release-1.1/arch-deep-dive.html#basic-workflow-of-transaction-endorsement).

### Single and Multi Deployment View of the SmartQuora DApp

The following diagram depicts a single-node view of the SmartQuora DApp.
![single node view](images/smartquora-arch-single-node-view.png)

The following diagram depicts the SmartQuora DApp when deployed on a multiple hosts to derive the benefits of the blockchain. For the sake of simplicity, in this tutorial, we will deploy the SmartQuora DApp as a logical instance on single host.
![enter image description here](images/smartquora-arch-multi-node-view.png)

## DApps - Decentralized Applications
### Understanding Blockchain
Before we break-down a DApp it is essential to understand its underlying technology - the Blockchain. Blockchain is a continously growing digital ledger of records organized in _blocks_ that are linked together by cryptographic validation. The key is to understand that this ledger is neither stored in a centralized location nor managed by any single entity, hence its *decentralized nature*. The block validation system results in new transactions being added irreversibly and old transactions preserved forever for all authorized participants to see, hence its transparency and resilience. Applications built on top of blockchain technology is called DApps. For an overview of blockchain and its underlying technologies, please review the following [whitepaper](http://bit.ly/blkchn2018). 
 
### Anatomy of the DApp (or BNA)
 In the world of Hyperledger Fabric, DApps are called BNA or Business Network Applications. We will use the Hyperledger Composer to model the SmartQuora BNA - Business Network Application.  Composer is an extensive, open development toolset and framework to make developing blockchain applications easier on the Hyperledger Fabric platform. Hence, I will use the term DApp or BNA interchangeably throughout the rest of the documentation.
![BNA Breakdown](images/bna-breakdown.png)

### Hyperledger Composer Components
Before we understand the components of the SmartQuora app, it is essential to understand the  Hyperledger Composer framework. A Hyperledger Composer consists of the following elements:

1. Participants
2. Resources comprising of Assets, Transactions, Events, Enumerated Types and Concepts.
3. Queries
4. Access Control Declarations.

- [ ] **Participiants** - Participants represents users who interact with assets.
- [ ] **Assets** - Assets represents entities which could represent place or things.
- [ ]  **Transactions** - Transactions are actions that participants can carry out on assets.
- [ ]  **Events** - Events are emitted by Hyperledger Composer as a result of transactions. 
- [ ]  **Queries** - SQL-like queries that can be used to search for assets based on its attributes.
- [ ]  **Access Control** - Access Control provides declarative access control over the elements of the domain model. By defining ACL rules you can determine which users/roles are permitted to create, read, update or delete elements in a business network's domain model.

## SmartQuora Domain Model
Now that we have an understanding of the components that make up a Hyperledger BNA, let us take a look at the components of the SmartQuora BNA. The following diagram provides a high-level view of the SmartQuora Domain Model.
![Domain Model](images/sq-process-flow.png)

### SmartQuora Participants
There are three types of Participants in SmartQuora BNA. They are: Inquirers, Responders and Administrators. Inquirers and Responders are represented as _QuoraUsers_ in the application because their function can interchange - an inquirer can respond to questions from other inquirers or a respondent for a question can pose his/her own questions. Participants maintain tokens that is placed as a stake when asking questions. 

The following rules are enforced by the system currently:
1. Inquirers cannot answer their own questions.
2. Responders cannot vote for their own answers.
3. Responders cannot vote more than once for the same answer.
4. Answers will not be accepted after the due date.
5. Only administrators can award questions and distribute the reward among the voted answers after the due date. 

### SmartQuora Assets
There are two types of Assets modeled in the SmartQuora application. They are: Question and Answer.

- [ ] **Question** - A question consists of an unique id, question description, owner, status (CREATED, ANSWERED, AWARDED, or DEFAULTED), reward amount and a list of answers. If a question is answered and is voted for, the stake is equally distributed among the owners of the voted answers.

- [ ] **Answer** - An answer consists of an unique id, answer description, owner, status (CREATED, VOTED, AWARDED), earnings, and a list of voters. When a question is awarded, the earnings attribute reflect the earnings that was generated by that particular answer for the respondent.

### SmartQuora Transactions
The following type of transactions are available in the SmartQuora application.

- [ ] **CreateQuestion** - A _CreateQuestion_ transaction is invoked by the Inquirer to pose a question. The reward amount and time by which answers are due has to accompany the request. The _CreateQuestion_ transaction generates the _QuestionCreated_ event.

- [ ] **CreateAnswer** - A _CreateAnswer_ transaction is invoked when a respondent provides an answer to an existing question. The _CreateAnswer_ transaction should be invoked with the identifier of the associated question and the answer description. The _CreateAnswer_ transaction generates an _AnswerCreated_ event. This transaction ensures that the question owners cannot answer their own questions.

- [ ] **VoteAnswer** - A _VoteAnswer_ transaction is invoked to vote up or vote down an existing answer. As a result, it should be invoked with a reference to an existing answer and the direction of the vote (up or down). A _VoteAnswer_ transaction generates the _AnswerVoted_ event. This transaction ensures that respondents cannot vote for their own answers or vote multiple tims for an answer.

- [ ] **AwardQuestion** - A _AwardQuestion_ transaction is invoked to find out the highest voted answers and distribute the reward proportionately amongst the voted answers. This transaction generates the _QuestionAwarded_ event.

### SmartQuora Events
Events in a Hyperledger Fabric BNA is pushed on a Web Socket. Any system listening to the Web Socket will get notified of all the events. As a result, the listeners have to filter out events that are not relevant to it.

SmartQuora events are as follows:
- [ ]  **QuestionCreated** - Generated by the _CreateQuestion_ transaction.
- [ ]  **AnswerCreated** - Generated by the _CreateAnswer_ transaction.
- [ ]  **AnswerVoted** - Generated by the _VoteAnswer_ transaction.
- [ ]  **QuestionAwarded** - Generated by the _AwardQuestion_ transaction.

## Setup Instructions
### Installing Hyperledger Fabric on AWS
#### Launch an EC2 Instance
For this exercise, we will deploy Hyperledger Fabric on a clean Ubuntu 16.04 LTS 64-bit environment on a AWS EC2 instance as per the configuration shown in the picture below. 
Make sure to open ports: 22, 80, 443, 8080-81 & 3000.
![AWS EC2 Instance Configuration](images/EC2%20Instance%20Configuration.png)
#### Associate an Elastic IP 
Allocate a new Elastic IP address and associate it with the newly created instance. This is not mandatory but will help maintain sanity while working on the scripts between relaunches of the instance.
#### Create Hyperuser
Hyperledger Fabric installation prohibits using the root identity to install the software. Therefore, create a new user and add that user to the _sudo_ group. You can choose any user name. I am using _hyperuser_

    sudo adduser hyperuser
    sudo adduser hyperuser sudo

Switch user to hyperuser and add the current working directory "." to the PATH environment variable in .profile.

    su - hyperuser
    vi .profile
    PATH=".:$HOME/bin:$HOME/.local/bin:$PATH"

**Note**: Always use _su -_ to switch user going forward to enable the environment variables are sourced from your _.profile_

**Note**: While this section goes over the installation of HLF on a compute instance on AWS EC2, the installation can be done on a compute instance on any public cloud such as Microsoft Azure, Google Cloud, IBM Bluemix or your own virtual machine on your local desktop/laptop.

#### Install the Hyperledger Fabric Pre-requisites
Use the following commands or follow the [installing pre-requisites guide](https://hyperledger.github.io/composer/latest/installing/installing-prereqs.html#ubuntu) to prep the environment for Hyperledger Fabric installation.
```bash
curl -O https://hyperledger.github.io/composer/latest/prereqs-ubuntu.sh

chmod u+x prereqs-ubuntu.sh
```
Next run the script to install the pre-requisites.
```
./prereqs-ubuntu.sh
```
After the installation is complete you should see a message stating that the following components are installed:

1. node
2. npm
3. docker
4. python

![enter image description here](images/pre-req-install-out.png)

__**At this point remember to log out and log back in to ensure you have access to the newly deployed binaries.**__

Verify that you have access to the binaries installed as follows:

    docker -v
    npm -v
    node -v
    python
    
#### Install the Hyperledger Development Environment
Follow the instructions below or use the [HLF Development Tools installation guide](https://hyperledger.github.io/composer/latest/installing/development-tools.html) to install the HLF development environment.

1.  Composer CLI tools:
    ```
    npm install -g composer-cli
    ```
2.  Utility for running a REST Server on your machine to expose your business networks as RESTful APIs:
    ```
    npm install -g composer-rest-server
    ```
3.  Useful utility for generating application assets:
    ```
    npm install -g generator-hyperledger-composer
    ```
4.  Yeoman is a tool for generating applications, which utilises  `generator-hyperledger-composer`:
    ```
    npm install -g yo
    ```
5. Install the Playground app used for editing and testing Business Networks:
	```
	npm install -g composer-playground
	```
6. Download the tool that installs Hyperledger Fabric:
	```
	mkdir ~/fabric-dev-servers && cd ~/fabric-dev-servers
	curl -O https://raw.githubusercontent.com/hyperledger/composer-tools/master/packages/fabric-dev-servers/fabric-dev-servers.tar.gz
	tar -xvf fabric-dev-servers.tar.gz
	```
7. Install Hyperledger Fabric runtime:
	```
	cd ~/fabric-dev-servers
	./downloadFabric.sh
	```

 8. Ensure five Docker images are downloaded:
	```
	docker images
	```
	You should see the following:
![enter image description here](images/docker-images-verification.png)

 9. The first time you start up a new runtime, you'll need to run the start script, then generate a PeerAdmin card which will be used to connect to the Hyperledger Fabric environment:
	 ```
	    cd ~/fabric-dev-servers
	    ./startFabric.sh
	    ./createPeerAdminCard.sh
	```
10. Verify that the Hyperledger Fabric environment started successfully by running the following:
	```
	docker ps
	```
	This should show four containers running which are the core of the Hyperledger Fabric environment.
	![HLF Started Verification](https://github.com/skarlekar/smart-quora/blob/master/images/hlf-started-verification.png)
12. You can also verify that your Hyperledger Fabric is functioning normally by interacting with it on the Playground web application. To do this, start the web application on port 8080 as follows:
     ```
	composer-playground 
	```
13. On your favorite browser, navigate to port 8080. If you see the following, your installation of Hyperledger Fabric runtime is successful.
![Playground starts](images/composer-playground.png)
 
 ### Starting and Stopping Hyperledger Fabric 
 1. To *start* the Hyperledger Fabric environment, use the following:
	 ```
	 cd ~/fabric-dev-servers
	 ./startFabric.sh
	 ```
 2. To *stop* the Hyperledger Fabric environment, use the following:
 	 ```
	 cd ~/fabric-dev-servers
	 ./stopFabric.sh
	 ```
## Creating a Hello World on Hyperledger Fabric
Now that we have installed Hyperledger Fabric runtime, let us create a Hello-World application to understand the principles and elements of Hyperledger Fabric.

1. In your home directory, use Yeoman to create a business application _hello_bna_ using:
```
	yo hyperledger-composer
```
Use the following instructions to respond to the menu options:
Select Business Network

> **Business network name**: hello-bna Description: My first BNA on HLF
> **Author name**: your-name 
> **Author email**: your-name@email.com 
> **License**: Press enter to accept Apache-2.0 
> **Namespace**: Press enter to accept default 
> **Do you want to generate an empty template network?**: Select No to generate a populated network

This will create the necessary code in the _hello_bna_ directory.

 2. Create a subdirectory called _dist_ inside the new _hello_bna_ directory and step into it.
 ```
mkdir dist; cd dist
```
 3. Create an archive by providing the type of archive as _dir_ and pointing to the parent directory.
```
composer archive create -t dir -n ../
```
4. Install version 0.0.1 of the business app using the peer admin card created earlier.
```
composer network install -a hello-bna@0.0.1.bna -c PeerAdmin@hlfv1
```
5. Create a business network admin card named *admin@hello-bna* by providing the *PeerAdmin@hlfv1* card and password *adminpw* to start version 0.0.1 of the app. This will also create the admin@hello-bna.card in the current directory.
```
composer network start  -A admin -S adminpw -c PeerAdmin@hlfv1 -n hello-bna -V 0.0.1
```
6. Import the admin@hello-bna.card using the composer card import command
```
composer card import -f ./admin@hello-bna.card
```
7. Ensure the new admin@hello-bna.card is imported using the composer-card-list command.
```
composer card list
```
This should display the existing card in your runtime as follows:
![composer card list output](images/composer-card-list.png)

8. Ensure that the application is running using the composer network ping command
```
composer network ping -c admin@hello-bna
```
9. Use docker ps to ensure that the docker container running the peer to serve the hello-bna application is running
```
docker ps
```
10. Open the BNA in Playground and connect with your application. Explore the model, participant, access control layer and transaction logic. 
```
composer-playground
```
11. Start the REST server & browse through the Swagger UI
```
composer-rest-server -c admin@hello-bna -n always -w true
```
Navigate to http://your-host-name:3000/explorer

## Installing SmartQuora
Now that we have seen a chance to take a sample application for a spin, it is time to install the SmartQuora application. 

1. Clone the SmartQuora Git on your host.
```
mkdir ~/projects
cd ~/projects
git clone https://github.com/skarlekar/smart-quora.git
```
2. Create a distribution directory.
```
cd smart-quora/smartquora-bna
mkdir dist
cd dist
```
 3. Create an archive by providing the type of archive as _dir_ and pointing to the parent directory.
```
composer archive create -t dir -n ../
```
3. Install version 0.0.1 of the business app using the peer admin card created earlier.
```
composer network install -a smartquora-bna@0.0.1.bna -c PeerAdmin@hlfv1
```
4. Create a business network admin card named *admin@smartquora-bna* by providing the *PeerAdmin@hlfv1* card and password *adminpw* to start version 0.0.1 of the app. This will also create the admin@smartquora-bna.card in the current directory.
```
composer network start  -A admin -S adminpw -c PeerAdmin@hlfv1 -n smartquora-bna -V 0.0.1
```
5. Import the admin@smartquora-bna.card using the composer card import command
```
composer card import -f ./admin@smartquora-bna.card
```
6. Ensure the new admin@smartquora-bna.card is imported using the composer-card-list command.
```
composer card list
```
This should display the existing card in your runtime as follows:
![composer card list output](images/sq-composer-card-list.png)

7. Ensure that the application is running using the composer network ping command
```
composer network ping -c admin@smartquora-bna
```
8. Use docker ps to ensure that the docker container running the peer to serve the hello-bna application is running
```
docker ps
```
9. Open the BNA in Playground and connect with your application. Explore the model, participant, access control layer and transaction logic. 
```
composer-playground
```
10. Start the REST server & browse through the Swagger UI. 
Note that we have started the REST server using Secure Socket Layer this time using the _start-resh.sh_ script. Hence, change the protocol from HTTP to HTTPS when browsing the REST Explorer. 
```
cd ..
./start-rest.sh
```
Navigate to https://your-host-name:3000/explorer

### Securing the REST Server
Now that we have a RESTful interface to our SmartQuora application, it is time to secure it using  the Passport Google OAUTH2.0 delegated authentication strategy.

![Google OAuth2.0 Authentication Strategy](images/Google%20OAUTH%20Overview.png)

While there are many Passport authentication strategy to choose from such as JWT, SAML, LDAP, AD etc, we will use Google+ API as the authentication provider for this exercise. The above diagram provides an overview of the authentication strategy. Here, the Composer REST server's role is to provide access to business network resources, which are protected by the Google+ API OAuth2.0 scheme. The resource owner is the Google+ API user account we set up.  Its role is to grant consent (or otherwise) to the client application. The Google+ authorization server requests consent of the resource owner and issues access tokens to REST clients  to enable them to access the protected resources. An access key is granted following consent in form of a token. This token allows a client to access the APIs protected by OAuth2.0.

In OAuth 2.0, these access tokens are called “bearer tokens”, and can be used alone, with no signature or cryptography, to access the information. Furthermore, the access token is stored in a cookie in the local storage of the user's web browser. When the user makes a subsequent request, the access token is retrieved from the cookie, and the access token is validated, instead of reauthenticating the user.

#### Google+ Authentication Configuration & Setup
1. Setup Google+ authentication using the guide [here](google-auth-setup.md).

2. Now copy the *client id* and *client secret* from Google+ and enter this in the _COMPOSER_PROVIDERS_ section of the  _start-smartquora.sh_ script.

3. Update the *successRedirect* field of the _COMPOSER_PROVIDERS_ with the host name of your web-server host (ec2 instance) in place of _your-host_.

4.  Now install the Passport Google OAuth2.0 strategy.
```
npm install -g passport-google-oauth2
```

#### Setting up the Credentials & Wallet Data Store using MongoDB
The REST Server itself is configured to persist the business network cards (required to connect to the network) using the MongoDB store. For this exercise we will use the MongoDB database hosted on http://mlab.com to keep it simple.

1. Point your browser to http://mlab.com and log into your account (create an account if you don't already have one).
![mlab login](images/sq-mlab-login.png)

2. Create a new database, select a cloud provider, select the free sandbox plan, click continue and select an available region. Click continue and  provide a database name as shown below. For this exercise we will  use a db called _quora-auth_.
![create database](images/sq-mlab-create-db.gif)

3. Create an user in the database. This user's credentials will be used by the REST server to connect to the database and use it for storing the credentials. For this exercise we will create a user called _test_ with password _test123_.
![create database user](images/sq-mlab-create-user.gif)

4. Now copy the database name and port from MLab and enter this in the _COMPOSER_DATASOURCES_ section of the  _start-smartquora.sh_ script.
![copy host port](images/sq-mlab-host-port.png)

5. Install the MongoDB Loopback connector plugin.
```
npm install -g loopback-connector-mongodb
```
### Start the REST server
After the changes from the above two sections, your _start-smartquora.sh_ should look like this:
![Startup Script](images/sq-start-smart-quora-sh.png)

Before you restart the REST server, shut it down if it is already running.
```
lsof -i :3000
```
This will give the process-id of the server attached to port 3000 (if it is running). 

Kill the process:
```
kill <pid>
```

Start the REST server by running the _start-smartquora.sh_ script.
```
cd ~/projects/smart-quora/smartquora-bna
./start-smartquora.sh
```

### Installing the Web Server 
1. For testing the BNA we will use a light-weight Node.js based web-server.  Install and start it as follows:
```
npm install -g http-server
```
2. Before we start the webserver, replace the 'your-host-name' string in the _html_ files with the host name of the machine where you have installed this software. Note: Only provide the domain name. Do not add the protocol or port here.
```
cd ~/projects/smart-quora/scripts
replace-host-name.sh ec2-sample-01.amazonaws.com
```
3. Start the web server.
```
cd ~/projects/smart-quora/www
http-server -p 8081
```
## Running SmartQuora
### Signing into SmartQuora 
Now, open your browser in **incognito** mode and point your browser to: http://your-host-name:8081/index.html
If everything went well, you browser should redirect to Google authentication page.
![sign-in page](images/sq-google-signin.png)

After authentication, Google Auth should redirect you back to the application. 

If you are doing this for the first time you will land in a page asking you to upload your digital identity to your wallet. 

Note that your wallet is maintained on the MongoDB database on your behalf by the REST server. If you check your MongoDB database now, you will see three collections viz., accessToken, user & userIdentity. But the collection for storing your Digital Identity called Card is not there yet as shown below
![mongodb collections before card](images/mongodb-collections-1.png)

### Creating Digital Identity Cards
We will use the Composer Playground to create a few digital identity cards as demonstrated in the animation below.
1. Start the Playground if it is not already up.
```
composer-playground
```
2. Point your browser to http://your-host-name:8080
3. Connect to the _smartquora-bna_ and click on _Test_
4. Create a new _Participant_ (make sure to fill out the emailId, fName, lName and token field. Give 100 tokens to start with).
5. Navigate to the _ID Registry_.
6. Click _Issue New ID_.
7. Provide a ID Name for the _Participant_.
8. Choose a _Participant_ from the list.
9. Click _Create New_ and choose _Send it to someone else_.
10. Click _Export Business Network Card_.

![create digital identity](images/create-digital-identity.gif)
Repeat this step a few times for multiple participants. I have created five participants.

### Adding the Digital Identity to your Wallet
 Now that we have created a few Digital Identities, let us import them into the application.
 1. Point your browser to http://your-host-name:8081/upload-card.html
 2. In the form that comes up, enter a name for the card and upload a card that was exported from Composer Playground.
 Note: Due to a bug in the upload-card code, you will not receive an acknowledgement of a successful upload.
 3. Repeat the step above for the remaining cards.
 ![Smart Quora Upload Card](images/sq-upload-card2.png)

### Selecting a Digital Avatar
Now that we have a few digital identities in our wallet, let us choose one to use.
1. Point your browser to home page of Smart Quora: http://your-host-name:8081/index.html
2. Click on the _Wallet_ button on the top left.
3. Choose a digital identity from the drop-down.
4. Click on _Change Card_ button.
5. You should get a message stating that the digital identity was changed to the card that was selected.
![wallet](images/sq-digital-wallet.png)

### Ask a Question
To ask a question:
1. Point your browser to home page of Smart Quora: http://your-host-name:8081/index.html
2. In the form _Get Answers To Your Own Questions_, enter a reward amount, due date for the question to be answered by in 24-hour format in the _Due_ field, your question in the _Ask A Question_ field and click the _Ask_ button.
3. If everything goes well, you should get a notification stating that the transaction was posted successfully and your question should pop-up in the list of questions above the form as shown in the picture below.
![question](images/sq-question1.png)

### Provide an Answer 
To answer a question:
1. Click on the question link in the home page of Smart Quora: http://your-host-name:8081/index.html
2. In the resulting page, enter your response in the _Your Answer_ section and click on the _Answer_ button.
3. If everything goes well, your answer should pop up in the list of answers above and you should get a notification that your transaction was posted successfully.
 ![question2](images/sq-question2.png)
 
### Vote for an Answer
To vote for an answer:
1. Click on the question link in the home page of Smart Quora: http://your-host-name:8081/index.html
2. In the resulting page, enter your click on the _thumbs up_ or _thumbs down_ button next to the answer. Please note that you cannot vote for your own answers or vote for an answer after the due date for the question.
3. If everything goes well, your vote should be accepted and the vote count should change accordingly. Additonally, you should get a notification stating that your transaction was posted successfully.
 ![answer](images/sq-answer1.png)
 
### Award an Answer
Awarding an answer is an administrator function and has to be performed in the back-end. Ideally, this will be triggered by a lambda function that finds all the question that are past current time and invokes the award transaction. For now, adding a query to retrieve past due questions and award them through a lambda function is left as an exercise to the reader.

To award an answer, on the server go to the _smartquora-gen_directory and run the _award_question.sh_ script. The script takes a question id as the only parameter.
```
cd ~/projects/smart-quora/smartquora-gen
award-question.sh your-question-id
```
![award](images/sq-award-question.png)

## Exercises
There are a few issues in this application as it stands. Can you fix them? Hints and solutions provided.

### Issue 1 - Dealing with _SpendThrifts_
The application allows a QuoraUser to spend indiscrimnately with total disregard to how much tokens they have in their account. Ideally, we want the QuoraUser to only offer up reward upto the amount they have in their account ie., QuoraUser.token
#### Solution 1
This can be easily solved by adding an access control rule that only allows a user to pose a question if they have atleast as much token as they are offering as a reward.

To do this, add the following to the _permissions.acl_ file.  
```
rule ThwartTokenlessQuestions {
  description: "Only Quora users with tokens can pose questions"
  participant(p): "smartquora.participant.QuoraUser"
  operation: CREATE
  resource(r): "smartquora.question.Question"
  transaction(tx): "smartquora.question.CreateQuestion"
  condition: (p.token < tx.offer)
  action: DENY 
}
```

Note: Rules are evaluated from top (most specific) to bottom (least specific). As soon as the Participant, Operation and Resource match for a rule then subsequent rules are not evaluated. If no ACL rule fires then the access control decision is DENY. Therefore, remember to add this rule around the top of the _permissions.acl_ file as shown [here](solution-1/permissions.acl).

Change the _permission.acl_ and upgrade the BNA as follows:
```
cd ~/projects/smart-quora
cp solution-1/permissions.acl smartquora-bna
cd smartquora-bna
node upgrade-bna.js
kill-rest.sh
start-smartquora.sh
```

### Issue 2 - Dealing with _DoubleSpenders_
While the _SpendThrifts_ problem is solved, it still leaves the problem of _Double Spending_ possible. This is where _Inquirers_ can rapidly create questions offering up stake before previous questions are awarded thereby depleting there coffers and leaving a negative balance in their accounts from future awards.

An additional issue is that, while you cannot update your own assets to enrich yourselves, you can update and enrich or impoverish another user by directly updating their corresponding _QuoraUser_ asset through direct CRUD operations.

#### Solution 2
One solution to this issue is to create a _Escrow_ account for each question which deducts the offer from the inquirer's account as soon as they pose a question. Once the question is awarded (or defaulted), the amount gets released from the Escrow account back to the awardee or the original inquirer (in case of default). 

By default when _permissions.acl_ exists, the policy is to deny everything. By adding the following rule _UpdateOnlyThroughTransactions_ instead of the existing _OwnerCannotUpdateTheirAssets_ we are allowing any transaction in the _smartquora_ name space to update our assets except where the users are the owner of the assets being operated on by the transaction. This will prevent users from performing CRUD operations and gaming the system 

```
rule UpdateOnlyThroughTransactions {
    description: "Assets can be updated only through transactions"
    participant(p): "smartquora.participant.QuoraUser"
    operation: UPDATE
    resource(r): "smartquora.**"
    transaction(tx): "smartquora.**"
    condition: (r.owner.getIdentifier() != p.getIdentifier())
    action: ALLOW
}
```
To use the _Escrow_ logic discussed above we have to add an _Escrow_ asset to the system. Additionally, logic must be added to subtract the stake from the user's balance and push it into a Escrow as soon as the question is created.

```
cd ~/projects/smart-quora/
cp solution-2/permissions.acl smartquora-bna
cp solution-2/question.cto smartquora-bna/models
cp solution-2/logic.js smartquora-bna/lib
cd smartquora-bna
node upgrade-bna.js
kill-rest.sh
start-smartquora.sh
```

> Written with [StackEdit](https://stackedit.io/).


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTgzNDkyNzE3NSwtNDQzODgxODQsMTAwNz
Q0NTgxMiw2MTAwOTkzMDcsLTEzNzU2NjI0NzEsMTg1NDgxNTMz
OSwxNTc4ODk4ODQ1LC0yMjkwMDkyOTYsMTUzNzExNDQxMywtMT
QxNzk1ODEwMiwtMTQ1MDAwMTM0LC04NjkzNjA3MjgsMTU4MDgy
NzI0NCwxMzMxNDc5NzQyLDQ0NjQyMzYyNywxMTgyOTk2NzcsNT
I0OTQ2MDcyLC0zMjg1NzI4NTMsLTY4MjAxMzU1MywzNTY0ODU0
NjhdfQ==
-->
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjQ5NDE4ODY3LDE0OTQ5NzUyNzAsLTIwMT
EyNjY5MDgsLTE2MTc1NzU0NjgsMTYxMzEyMTkyMCwxNDYzNTY5
MTA5LC0xNDU1NjY2ODAzLDEzNDkyMzUxOTIsLTgxNjM4MTMwMy
wtMTA2MTUyNzI2NV19
-->
