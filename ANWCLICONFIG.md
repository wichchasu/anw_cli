### 1. **สร้างบัญชี AWS**
1. ไปที่ [AWS](https://aws.amazon.com) และสร้างบัญชี

### 2. **ติดตั้งเครื่องมือที่จำเป็น**
1. **AWS CLI**: ติดตั้ง AWS CLI เพื่อใช้งาน AWS จาก command line
   - Mac: `brew install awscli`
   - Windows: ดาวน์โหลดจาก [ที่นี่](https://aws.amazon.com/cli/)
   - เมื่อติดตั้งเสร็จ ให้รันคำสั่ง: `aws configure` แล้วกรอก Access Key, Secret Key, Region ตามบัญชี AWS ของคุณ
2. **kubectl**: ติดตั้งเครื่องมือสำหรับจัดการ Kubernetes cluster
   - Mac: `brew install kubectl`
   - Windows: ดาวน์โหลดจาก [ที่นี่](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
3. **Terraform**: ติดตั้ง Terraform
   - Mac: `brew install terraform`
   - Windows: ดาวน์โหลดจาก [ที่นี่](https://www.terraform.io/downloads)

### 3. **สร้างโปรเจกต์ Next.js หรือ React**
1. เปิด Terminal หรือ Command Prompt แล้วพิมพ์คำสั่งต่อไปนี้:
   - **สำหรับ Next.js**: 
     ```bash
     npx create-next-app@latest my-next-app
     cd my-next-app
     npm run build
     ```
   - **สำหรับ React**:
     ```bash
     npx create-react-app my-react-app
     cd my-react-app
     npm run build
     ```
2. คำสั่งนี้จะสร้างไฟล์ที่พร้อมสำหรับการ deploy ในโฟลเดอร์ `build` ของโปรเจกต์.

### 4. **สร้างไฟล์ Terraform สำหรับ EKS**

1. สร้างโฟลเดอร์ใหม่สำหรับเก็บไฟล์ Terraform
   ```bash
   mkdir my-eks-terraform
   cd my-eks-terraform
   ```

2. สร้างไฟล์ `main.tf` สำหรับกำหนดโครงสร้างพื้นฐาน EKS ของคุณ โดยใช้โค้ดตัวอย่างนี้:

```hcl
provider "aws" {
  region = "us-west-2"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  count = 2
  vpc_id = aws_vpc.main.id
  cidr_block = cidrsubnet(aws_vpc.main.cidr_block, 8, count.index)
  availability_zone = data.aws_availability_zones.available.names[count.index]
}

resource "aws_eks_cluster" "my_cluster" {
  name     = "my-cluster"
  role_arn = aws_iam_role.eks_role.arn

  vpc_config {
    subnet_ids = aws_subnet.public[*].id
  }
}

resource "aws_iam_role" "eks_role" {
  name = "eks-role"

  assume_role_policy = jsonencode({
    "Version" : "2012-10-17",
    "Statement" : [{
      "Action"    : "sts:AssumeRole",
      "Effect"    : "Allow",
      "Principal" : {
        "Service" : "eks.amazonaws.com"
      }
    }]
  })
}
```

3. **จัดการกับ IAM และ Security Groups**: คุณจำเป็นต้องเพิ่ม IAM role และ Security Group สำหรับ EKS และ Node Group ของคุณใน Terraform configuration.

### 5. **รัน Terraform เพื่อสร้างโครงสร้างพื้นฐาน**

1. เริ่มต้นด้วยการ initialize Terraform:
   ```bash
   terraform init
   ```

2. ตรวจสอบ plan ว่าโครงสร้างพื้นฐานจะถูกสร้างขึ้นอย่างไร:
   ```bash
   terraform plan
   ```

3. ถ้าทุกอย่างถูกต้อง ให้ deploy โครงสร้างพื้นฐาน:
   ```bash
   terraform apply
   ```
4. เมื่อโครงสร้างพื้นฐานถูกสร้างขึ้น คุณจะได้รับข้อมูลเกี่ยวกับ EKS cluster และ Kubernetes API URL.

### 6. **เชื่อมต่อกับ EKS Cluster**

1. ใช้ `kubectl` เพื่อตั้งค่า context ให้เชื่อมต่อกับ EKS cluster ที่คุณสร้าง:
   ```bash
   aws eks --region <your-region> update-kubeconfig --name my-cluster
   ```

2. ตรวจสอบว่า `kubectl` เชื่อมต่อกับ EKS cluster สำเร็จ:
   ```bash
   kubectl get svc
   ```

### 7. **สร้างและ deploy Docker Image ของแอป Next.js หรือ React**

1. ใน root ของโปรเจกต์ Next.js หรือ React ของคุณ สร้างไฟล์ `Dockerfile`:
   ```Dockerfile
   FROM node:14-alpine

   WORKDIR /app

   COPY package.json yarn.lock ./
   RUN yarn install

   COPY . .

   RUN yarn build

   EXPOSE 3000
   CMD ["yarn", "start"]
   ```

2. สร้าง Docker image สำหรับแอปของคุณ:
   ```bash
   docker build -t my-app .
   ```

3. Push image ไปที่ Amazon Elastic Container Registry (ECR) หรือ Docker Hub:
   ```bash
   aws ecr create-repository --repository-name my-app
   $(aws ecr get-login --no-include-email --region us-west-2)
   docker tag my-app:latest <your-account-id>.dkr.ecr.us-west-2.amazonaws.com/my-app:latest
   docker push <your-account-id>.dkr.ecr.us-west-2.amazonaws.com/my-app:latest
   ```

### 8. **Deploy แอปบน EKS**

1. สร้างไฟล์ `deployment.yaml` เพื่อ deploy แอปของคุณบน EKS:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-app
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: my-app
     template:
       metadata:
         labels:
           app: my-app
       spec:
         containers:
         - name: my-app
           image: <your-account-id>.dkr.ecr.us-west-2.amazonaws.com/my-app:latest
           ports:
           - containerPort: 3000
   ```

2. Apply deployment นี้ไปที่ EKS:
   ```bash
   kubectl apply -f deployment.yaml
   ```

3. สร้าง Service เพื่อ expose แอปของคุณ:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: my-app-service
   spec:
     type: LoadBalancer
     selector:
       app: my-app
     ports:
     - protocol: TCP
       port: 80
       targetPort: 3000
   ```

4. Apply Service:
   ```bash
   kubectl apply -f service.yaml
   ```

### 9. **เข้าถึงแอป**
หลังจากที่คุณสร้าง Service แล้ว คุณสามารถดู LoadBalancer URL ที่ถูกสร้างขึ้นโดย AWS เพื่อเข้าถึงเว็บแอปของคุณ:
```bash
kubectl get svc my-app-service
```

จะมี URL ของ LoadBalancer ปรากฏขึ้น ซึ่งคุณสามารถใช้เพื่อเข้าถึงเว็บแอปของคุณทางออนไลน์!

---

### สรุปขั้นตอน:
1. สร้างบัญชี AWS และติดตั้งเครื่องมือที่จำเป็น.
2. สร้างโปรเจกต์ Next.js หรือ React.
3. เขียนไฟล์ Terraform เพื่อสร้าง VPC, Subnets, และ EKS.
4. ใช้ `terraform apply` เพื่อสร้างโครงสร้างพื้นฐาน.
5. เชื่อมต่อกับ EKS ด้วย `kubectl`.
6. สร้าง Docker image และ push ไปที่ ECR.
7. Deploy แอปของคุณบน EKS ด้วย `kubectl`.
8. ใช้ LoadBalancer เพื่อตั้งค่า URL สำหรับการเข้าถึงแอป.

