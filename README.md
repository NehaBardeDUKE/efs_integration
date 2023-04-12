# EFS-LAMBDA integration

This project details the EFS-Lambda integration process, made robust by many trial and errors-

## Create efs :
1. Use the standard configuration. Nothing special. By default there will be a VPC selected, change it if necessary but make sure that any resources that need to communicate with efs are part of the same VPC

![image](https://user-images.githubusercontent.com/110474064/231590377-abc06b57-2dd9-4d1c-89b1-65a8b3697439.png)

2. When you click on "create" it also by default creates a security group say S1.

![image](https://user-images.githubusercontent.com/110474064/231591224-046c6658-a207-462f-9066-bf8268479584.png)

![image](https://user-images.githubusercontent.com/110474064/231591562-a18e867a-dcb0-4085-a570-470462e35652.png)

3. Create an access point. Note: do not populate the directory here. it will interfere with the lambda fs settings when lambda tries to mount
basically no need to populate anything that is optional for basic functionality

![image](https://user-images.githubusercontent.com/110474064/231592026-c2479b5e-ceed-42d1-b6a9-2acde0349e0e.png)

## Mount efs:
1. to mount the efs, the request for the efs component is outbound in nature but for EC2 (the place where u want to mount it) is inbound in nature. So for EC2 you will have to update inbound rules to allow S1 on 2049. Security grps are how the different components talk to each other.

2. To do this go to EC2 , scroll down and from the list of services on the left, select security grps.

3. Look for the S1 grp (this is the grp that needs to be given the access). Scroll down , you will see inbound rules , click on edit and add a new rule. Now the security grp to be added should be the cloud9 security grp u wish to mount the efs on.(the below circled security grp are for mu cloud9 resource called "Rust-deploy". I basically just searched for the resource name instead of looking for the security grp explicitly)

![image](https://user-images.githubusercontent.com/110474064/231593347-3036bf74-1008-4234-9c60-f455681a2d81.png)

4. Now open up the cloud9 resource and while that is loading go back to EFS , click on the EFS u just created and you will see an option called "attach". 

![image](https://user-images.githubusercontent.com/110474064/231593781-e36b1bcc-0340-4db1-a650-b3dfb5d8443f.png)

Click on that. it will give you a set of commands that u can use to mount the efs onto a VM (EC2) using cloud9
I used the mount helper and mounted using DNS

![image](https://user-images.githubusercontent.com/110474064/231594007-5f1874d9-81c9-494a-853e-139c055bf934.png)

Copy the command and just run it on your Cloud9. (Note: 1 small change the command says sudo mount -t efs -o tls fs-0133a14a44dac09bf:/ efs , replace the "efs" at the end with the mount point you want. I did it as sudo mount -t efs -o tls fs-0133a14a44dac09bf:/ /mnt/efs)

## Create Lambda:

Note: before creating lambda make sure to create the IAM role and give it the right permissions. This role will be used to create lambda as below. If u are doing this through UI (using ECR) there will be a place where it asks you to put the role. If you are doing this using cli, u will need the role, or it wont let u deploy lambda.
I created a IAM role (lambdaefs) with Admin permissions (not ideal but just did it for the POC)

1. You can create lambda using the cargo-lambda library and push out required function.

2. I already had the image on docker hub, so i just pulled it to my cloud 9 and pushed it to an ECR repo.

3. then created lambda using ECR image. 

![image](https://user-images.githubusercontent.com/110474064/231595615-4e487ea1-a453-4608-8faf-1792d25a5420.png)

![image](https://user-images.githubusercontent.com/110474064/231595869-409423d3-2993-43de-8e27-6fcab1ebbc15.png)

4. I actually also ended up creating one from scratch but both methods ultimately give u the same thing, the behavior of the EFS mount doesnt change wrt the method of lamda creation.

CLI: cargo lambda deploy --iam-role arn:aws:iam::<your acc.id>:role/lambdaefs --region us-east-1 --verbose


## Lambda settings:

Note: make sure that if u didnt have an IAM role at the time of creating the lambda, u do it now and update your lambda to use that role for executions, or u wont be able to proceed with the next steps.

1.VPC : Go to the lambda function u created and click on configuration and then find VPC. When u click on it, u will see nothing set up. Click on edit and select the VPC of your EFS and select all subnets. It will also ask u to select the security grp. select your efs's security grp

![image](https://user-images.githubusercontent.com/110474064/231596933-d0bceb06-98c6-4fab-abdb-15637896427f.png)

2. File Systems: This tells the lambda to go to the required mount point. When u configure the file systems, it will ask u for the efs, for the access point and the absolute director structure. Select the efs, and the access point u created and the absolute directory structure is nothing but your mount point.

![image](https://user-images.githubusercontent.com/110474064/231597373-56a886c1-ce98-488d-8fd6-239f8a698648.png)

THAT'S IT!!

## Testing:
Now just fire some test cases on your lambda and u should see a successful execution if everything works right. My funtion lists the files in the EFS dir

![image](https://user-images.githubusercontent.com/110474064/231597617-cddbddbb-27b6-4bc2-80be-3d0ae8bddf2d.png)
![image](https://user-images.githubusercontent.com/110474064/231597782-0b96b63e-e886-4709-b4a4-713afb460793.png)

Actual code is part of an individual project- repo: https://github.com/NehaBardeDUKE/Project4_with_Rust





