# Internet-Protocols---Public-and-Private-IP-addresses
Troubleshooting EC2 Internet Connectivity &amp; CIDR Best Practices




## **Project Overview**

I analyzed a real-life case scenario where a Fortune 500 company faced a networking issue within their AWS infrastructure. The setup included a VPC with a CIDR range of 10.0.0.0/16 and two EC2 instances, but only one had internet access. Additionally, the scenario posed a question about using a public CIDR range for a new VPC.

<img width="832" alt="Screenshot 2025-03-22 at 12 11 40" src="https://github.com/user-attachments/assets/24e0db80-6510-4e61-9de5-20e961ce39a5" />

Image: customers architecture

## **Step 1: Investigating the Environment**


As a cloud support engineer in this scenario, I received a ticket from a customer who reported that one of their EC2 instances could not reach the internet while another instance in the same subnet had no issues. 

The customer's architecture was as follows:
- **VPC CIDR:** 10.0.0.0/16
- **Instance A:** Could not access the internet.
- **Instance B:** Could access the internet.

I accessed the AWS Management Console and navigated to the EC2 dashboard. There, I examined the networking configurations of both EC2 instances.

<img width="1439" alt="Screenshot 2025-03-20 at 01 05 57" src="https://github.com/user-attachments/assets/937b713a-dba5-4a78-a314-d4222316152f" />

---

- **Instance A:** Only had a private IP address.
  <img width="1440" alt="Screenshot 2025-03-20 at 01 10 22" src="https://github.com/user-attachments/assets/9c5c51e6-9ed4-4886-bbf3-b604ee584847" />
---

- **Instance B:** Had both a private and a public IP address.
  <img width="1440" alt="Screenshot 2025-03-20 at 01 11 12" src="https://github.com/user-attachments/assets/d304b272-cad6-4b25-9951-6af196bae7c1" />


---
- **Observation:** The difference in internet access stemmed from the presence or absence of a public IP address.

## **Step 2: Using SSH to Connect to EC2 Instances**
To further analyze the issue, I attempted SSH access to both instances running the following command: 


   ```
   # InstanceA
   cd ~/Downloads
   
   ls -l labsuser.pem

ssh -i labsuser.pem ec2-user@10.0.10.226

   ```

  ```
# Instance B
   cd ~/Downloads
   
   ls -l labsuser.pem

ssh -i labsuser.pem ec2-user@34.219.185.136

   ```

- **Instance B (Public IP Address):** Successfully connected via SSH.
- **Instance A (Private IP Address):** Unable to connect from an external network due to the lack of a public IP.
- **Conclusion:** Since instance A only had a private IP, it was unreachable from outside the VPC. This confirmed that the absence of a public IP was the root cause of its internet connectivity issue.

## **Step 3: Addressing the Public CIDR Range Concern**
The customer also asked about using a public IP range (e.g., 12.0.0.0/16) for a new VPC. Based on best practices:

- Public IP ranges (e.g., 12.0.0.0/16) are part of globally routable IPs, typically assigned by Internet Service Providers (ISPs). AWS does not allow the use of public IP ranges within a VPC because it could lead to conflicts and routing issues.
- Instead, private IP ranges defined by RFC 1918 should be used:
  - **10.0.0.0/8**
  - **172.16.0.0/12**
  - **192.168.0.0/16**

## **Step 4: Implementing a Solution**
To restore internet connectivity for Instance A, I outlined two possible solutions:

1. **Assign a Public IP Address**: This would enable direct internet access, but it would require a security group update to allow necessary inbound/outbound traffic.
2. **Use a NAT Gateway**: This option allows private instances to access the internet without exposing them directly. It involves setting up a NAT Gateway in a public subnet and updating route tables accordingly.

Given the security considerations, I recommended the NAT Gateway solution for a more secure and scalable setup.



## **Step 5: Formulating a Response**
After gathering all findings, I drafted a response to explain the issue and solutions:

---

**Subject:** Resolution for Internet Connectivity Issue and CIDR Range Inquiry

Hello Cynthia,

After reviewing your setup, I identified the following:

- **Instance A lacks a public IP address**, preventing it from accessing the internet.
- **Instance B has a public IP address**, enabling internet access.

To allow instance A to reach the internet, you can assign it a public IP or set up a NAT gateway if private instances require outbound internet access.

Regarding the question about using a public CIDR (12.0.0.0/16) for a new VPC:

- AWS does not support using public IP ranges for VPC CIDR blocks.
- Using public IPs within a VPC can cause conflicts with global routing.
- Instead, it is recommended to use private IP ranges as defined in RFC 1918 (e.g., 10.0.0.0/16).

Setting up a NAT Gateway is a secure way to provide internet access to private instances. Let me know if further guidance is needed.

Best regards,  
Dolapo Babs.

---

This response clarified the issue while offering solutions and explaining best practices for VPC CIDR allocations.






