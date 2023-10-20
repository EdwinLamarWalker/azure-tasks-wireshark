![image](https://github.com/EdwinLamarWalker/configure-VM/assets/147763790/c1fc6f41-95af-40fb-8c7c-f638843c7c1a)


<h1>Microsoft Azure - Tasks In Azure using wireshark and Powershell </h1>
In this Tutorial we will be observing icmp traffic and performing task inside azure's virtual machines using Wireshark and Powershell.<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Wireshark
- Powershell
- Network Security Group (Azure Firewall)
  

<h2>Operating Systems Used </h2>

- Windows 10</b> (21H2)
- Linux (ubuntu 20.04)
<h2>List of Prerequisites</h2>

- Have 2 Virtual Machines up In Azure (Windows and Linux.)
- Download Wire shark and open powershell in windows virtual machine.
- Send icmp traffic in the Powershell command-line.
- Access Inbound NSG(Network security group) And Deny Traffic.
- Item 4
- Item 5

<h2> Steps</h2>

![image](https://github.com/EdwinLamarWalker/azure-tasks-wireshark/assets/147763790/6b2dee65-325e-4cc7-9cea-8c47874b980e)

<p>
Before Opening up your virtual machine, we must first check our resource group to check a couple of things. First to see if your vm1 and vm2 are created in the same RG(Resource Group.) Then to verify they were put under the same network group as well as the same region. This is important for if your Virtual machines arent under the same virtual network the private Ip Address they have will be the same and not different (You wont be able to send traffic from your vm1 toward your vm2.) </p>
<br />

![image](https://github.com/EdwinLamarWalker/azure-tasks-wireshark/assets/147763790/392d2cd8-9647-425c-9bc0-0069c9dfdf76)

Once You have went and copied your vm1 (Windows Virtual Machine Ip) you search remote desktop in windows search bar and Paste that into it. The same username and password you created from your last lab will be used to sign into your vm. This Would be the same as signing into a customers computer remotely using their Ip address and infromation given.
</p>
<br />

![image](https://github.com/EdwinLamarWalker/azure-tasks-wireshark/assets/147763790/c629e4c8-658a-440d-bf7a-7743d5a13839)
As for the next step, in your virtual machine search and install wireshark. This is what we use to observe the traffic from our vm.


![image](https://github.com/EdwinLamarWalker/azure-tasks-wireshark/assets/147763790/953916f7-9f60-4889-9a7b-9200a962dfd9)
![image](https://github.com/EdwinLamarWalker/azure-tasks-wireshark/assets/147763790/8f17253a-f665-42fc-9168-752dbb0749c5)

 Once you get wireshark installed search powershell and open the two together, organize to fit screen. They both should look blank when opened. Then to connect to our vm we press ethernet and then press the blue wireshark symbol on the top left, it will then start to show traffic because we are running our virtual machine.

![image](https://github.com/EdwinLamarWalker/azure-tasks-wireshark/assets/147763790/5deb22d5-1a89-462d-8c3b-12a2aa9689ce)


The first thing we do is filter for icmp traffic since there isnt any and we havent entered anything inside of the powershell command line it should be blank. From here you want to go back and Jot down both of your vm1 and vm2 private ip addresses which in my case is 10.0.0.5 and 10.0.0.4. This will be important because we will use the private ip addresses to send traffic.

![image](https://github.com/EdwinLamarWalker/azure-tasks-wireshark/assets/147763790/17f03ab3-b874-4b22-997f-72a09370b12a)

What I did here was send a ping request to my vm2 (10.0.0.5) from my vm1 (10.0.0.4.) If you look on my powershell I Sent 4 packets, received 4 packets, and lost none. So my vm2 responded 4 times in total from the packets I sent.


![image](https://github.com/EdwinLamarWalker/azure-tasks-wireshark/assets/147763790/b560d795-31e3-456f-abcd-d1f7bb2857dc)

This Time, Instead of pinging my vm2, i pinged google in -4 forcing it to use ipv4 instead of ipv6. This traffic is interesting because we are actually pinging and recieving from one of googles public ip addresses, and Since we arent really sending anything all its sending is junk data.


![image](https://github.com/EdwinLamarWalker/azure-tasks-wireshark/assets/147763790/e55529e8-df54-476a-aed8-dadaeabf0b35)

Now what I did here was initiate a purpetual ping from 10.0.0.4 to 10.0.0.5. Since its a ping that constantly requests we will enter the firewall of our vm2 (10.0.0.5) and deny the pings access.


![image](https://github.com/EdwinLamarWalker/azure-tasks-wireshark/assets/147763790/b58f9235-a29f-4851-be78-3546c33f58c1)

These are the rules that already exist,The # priority is when traffic comes into vm2 its checked against the rules and the action with those rules happen. The SSH rule above is for tcp port 22 and if there is tcp traffic over that port (Or Any Destination) the action is "allowed." So with that being said we are going to create a rule that "DENY" any tcp traffic to vm2, then observe it.


![image](https://github.com/EdwinLamarWalker/azure-tasks-wireshark/assets/147763790/023ca85d-27e6-4000-ab1a-dc4671d0fca6)

If You look closely, you will see that we are creating a rule for any icmp traffic since it doesnt have a port. The priority is important because thats the action that is taken first, so our deny priority has to be any number lower than 300 since thats what the tcp port 22's priority is. Once created, this rule will stop the purpetual ping to vm2.


![image](https://github.com/EdwinLamarWalker/azure-tasks-wireshark/assets/147763790/d15ea0e9-8b9c-466f-8f49-e0c0a78be675)

Instantly the purpetual ping starts to "timed out" this is because of the rule we set with a higher priority then the tcp port 22 rule. We have went into vm2's firewall and stopped any icmp traffic from reaching our vm2. If you look at our wireshark we are sending responses from our vm1 to vm2 and getting no response. Since the packets we sent arent reaching like before. 

![image](https://github.com/EdwinLamarWalker/azure-tasks-wireshark/assets/147763790/0b62645b-3f4f-49c6-8a12-3ba58b4ee9a5)

![image](https://github.com/EdwinLamarWalker/azure-tasks-wireshark/assets/147763790/9635646c-ca0e-4368-af57-99da636d1b9f)

Now to stop this rule you can simply delete the rule, or you can go inside vm2 firewall and edit the rule to "allow" icmp traffic and it will go back to receiving traffic to our vm2. Crtl+C to stop the -t ping.

