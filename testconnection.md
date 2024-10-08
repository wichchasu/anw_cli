### วิธีทดสอบการเชื่อมต่อ AWS CLI:

1. **เปิด Terminal** บนเครื่อง Mac ของคุณ.
   
2. รันคำสั่งนี้เพื่อทดสอบการเชื่อมต่อ:
   ```bash
   aws sts get-caller-identity
   ```

   คำสั่งนี้จะตรวจสอบตัวตนของคุณใน AWS และแสดงรายละเอียดของผู้ใช้ที่กำลังใช้งาน (เช่น AWS Account ID, User ARN เป็นต้น).

3. คุณจะเห็นผลลัพธ์คล้ายๆ ดังนี้:
   ```json
   {
       "UserId": "AKIAIOSFODNN7EXAMPLE",
       "Account": "123456789012",
       "Arn": "arn:aws:iam::123456789012:user/your-username"
   }
   ```

   ถ้าคุณเห็นข้อมูลผู้ใช้ในลักษณะนี้ แสดงว่าคุณเชื่อมต่อกับ AWS ได้สำเร็จแล้ว.

### การทดสอบคำสั่ง AWS CLI อื่นๆ:
- ตรวจสอบ EC2 instances:
   ```bash
   aws ec2 describe-instances
   ```

- ตรวจสอบ S3 buckets ที่มีอยู่ในบัญชีของคุณ:
   ```bash
   aws s3 ls
   ```

ถ้าคำสั่งเหล่านี้ทำงานและแสดงข้อมูล แสดงว่าการเชื่อมต่อระหว่าง AWS CLI และบัญชี AWS ของคุณทำงานได้ถูกต้อง
