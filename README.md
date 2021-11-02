# Object specific versioning in AWS S3 bucket

This is my first blog ever. The idea of the blogging on Versioning of S3 Bucket came through my personal experience and what I did to overcome the challenges. So before going forward let me list down the topics I will be covering in this blog. 
1. A brief introduction of AWS S3 Bucket
2. Prerequisite
3. Versioning in S3
4. Challenge of Object specific versioning in S3 bucket
5. Solution!!
6. Summary

## 1. A brief introduction of AWS S3 Bucket
Amazon Web Services (AWS) is a cloud platform that provides tremendous cloud services. These services allow users to concentrate more on the development of an application rather than resources/environment as that will be handled by AWS. One of the very famous services of AWS is AWS S3( Simple Storage Service).

*Amazon Simple Storage Service (Amazon S3) is an object storage service that offers industry-leading scalability, data availability, security, and performance. Amazon S3 provides management features so that you can optimize, organize, and configure access to your data to meet your specific business, organizational, and compliance requirements.* [1](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)

AWS S3 provides many functionality which you can check on it’s original website. Here we will only focus on the ‘Versioning of S3 Buckets’. AWS S3 is a storage service where we can store any kind of data/file in the form of an object without worrying much about its management. S3 refers to any data/file as an Object and these objects must be stored in a specific Bucket. 
So before storing any object in a bucket you must create a bucket first if not present. Bucket is a container which stores a number of objects in it. 

Every object has its own unique identifier known as Key. The combination of a bucket, object key, and optionally, version ID (if S3 Versioning is enabled for the bucket) uniquely identifies each object.  
These Keys basically look like a file path, for example “reports/Sales_Records_2021.xlsx” is the name of a key in a bucket “Bucket-Sale”. There can’t be two duplicate Keys in a Bucket. 

This is a brief introduction of the AWS S3 bucket. As this session is concentrated on challenges faced in Versioning of AWS S3 bucket so we will move forward with our topic. To get more insight of the topic you can visit AWS S3 official website.[1](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)

## 2. Prerequisite 
To access AWS S3 service you must have an account on AWS with IAM user role. To access outside its environment you will need AWS Access Key ID and AWS Secret Access Key. Then you are good to go. 

## 3. Versioning in S3 
As the title suggests, “Versioning in S3” means adding version to the objects of a bucket. AWS S3 gives privilege to add multiple versions of an object. The motive behind this is to  save all the previous versions of an object. It is like saving the older version as a backup. This is very helpful when the current version is not working properly or can recover if any accident occurs. 

*When you enable S3 Versioning in a bucket, Amazon S3 generates a unique version ID for each object added to the bucket. Objects that already existed in the bucket at the time that you enable versioning have a version ID of null. If you modify these (or any other) objects with other operations, such as CopyObject and PutObject, the new objects get a unique version ID.* [2](https://docs.aws.amazon.com/AmazonS3/latest/userguide/versioning-workflows.html)

## 4. Challenge of Object specific versioning in S3 bucket
Now you know that versioning is added to all the objects of a bucket (version enabled). But what if we don’t want to add versioning to all the objects in a bucket. There can be some cases where we only want versioning on some of the files and the rest should overwrite. 

Also if we want to get versioning for a few of the files still we have to pay the complete price of a bucket versioning. During my working experience I encountered such a scenario. I have to store some of the files in an already existing S3 bucket with a versioning feature and this bucket didn’t have versioning enabled. But as a developer I have to figure out some solution to this. So move to the next section to get an alternative to this issue. 

## 5. Solution!!
There is no direct solution to the mentioned problem. Instead we have to do it manually. So let's take a scenario and move forward with it to get a better understanding.

We will take 3 scenarios:
1. No versioning on the objects. 
2. Separate versioning on every object.
3. Same version number on a set of objects. 

### *Scenario 1: No versioning on the objects*
Here the version ID will be null for all the objects so the objects will be replaced with an existing object. 
Old object key: *S3_no_version/object_with_null_version.pdf*
New object key: *S3_no_version/object_with_null_version.pdf*

Note: Ignore the file type, it can be of any type (xlsx, py, text, jpg, etc.)

### *Scenario 2: Separate versioning on every object*
Let's consider we have two objects in a Bucket called S3_Demo_Bucket
Object 1 key: *S3_with_version/demo_object_1.pdf *
Object 2 key: *S3_with_version/demo_object_2.pdf *

Now you want to add versions to both of the objects but they are not necessarily the same. This means that these both objects are irrelevant to each/ no need to follow same version. 

In this case you need to keep an account on the current version or can do it using date and time, whichever satisfies your condition.
Some of the examples of adding version to an object are as follows:
1. Object_name_v_01.pdf
2. Object_name_281020211007.pdf    // appended with current date and time
3. Object_name_1.pdf

Here, v_01 and _1 is the version number. You can store this version number in a variable and then sequentially increase it when an updated object comes. 
Note: it will be hard to add versions in fractions (like 1.2, 23.9, etc.) as 

A sample code is given for your reference (in python): 
```
# An object key can be in any format so let's take folder_name/object_name_v_54.py as the sample key. You can set the version number as per your convenience. 

class S3Versioning():
    def __init__(self):
        pass

    def get_current_version(self, object_key):
        try: 
            file_name = object_key.split('/')[-1]
            print('Old file name = ' + file_name)
            self.version_number = int(file_name.split('.')[-2].split('_')[-1])     #version number is mentioned in the last three places 
        except:
            self.version_number = 0
        print('Current Version Number = ' + str(self.version_number))
    
    def update_current_version(self, object_key):
        self.get_current_version(object_key)
        self.version_number = self.version_number + 1
        
        file_name = object_key.split('/')[-1]
        separator = '/'
        new_object_key =  separator.join(object_key.split('/')[:-1]) + separator + '_'.join(file_name.split('.')[-2].split('_')[:-1]) + '_' + str(self.version_number) + '.py'
        
        print('New object key = ' + new_object_key)
    
s3_version_obj = S3Versioning()
s3_version_obj.update_current_version("bucket_name/folder_name/object_name_v_54.py")
```
Output:
```
Old file name = object_name_v_54.py
Current Version Number = 54
New object key = bucket_name/folder_name/object_name_v_55.py
```

The above can be modified and updated as per your requirement. Also, this sample code is not the best solution, you can try your own version and share with all. 

Now here you can see that lots of string separation and joins are working together to get the desired result and it will be not efficient if we want some set of files to follow the same version. The solution to this problem is covered in the next section. 

### *Scenario 3: Same version number on a set of objects*
There are some cases where we want to set the same version number on a set of objects. 

For example: If you want to save your publications file in S3 and the object key looks like “Publication_list/Plants_growth.docx” and you also want to add a statistic file with key name “Publication_list/Plants_growth_statistics.xlsx” which has statistics described/mentioned in the Plants_growth.docx file then every time you upgrade your statistics  you also have to made changes in your publication document. In this case you have to keep the same version number on both of the files. 
Here’s the solution. Instead of appending the version number in your file name it is better to add the version number in your object key. 
Like: Publication_list/1/Plants_growth_statistics.xlsx and Publication_list/1/Plants_growth.docx
1 is the version number and you can get this number without disturbing the file name. This method is very efficient in such a case as it manatin the same number on all the relevant files and also its retrieval is also easy compared to previous method. It is not efficient to apply in the previous section scenario. 

Sample code is given below. 
```
class S3CombineVersioning():
    def __init__(self):
        pass

    def get_current_version(self, object_key):
        try: 
            self.version_number = object_key.split('/')[-2]
        except:
            self.version_number = 0
        print('Old Version Number = ' + str(self.version_number))
    
    def update_current_version(self, object_key):
        self.get_current_version(object_key)
        self.version_number = int(self.version_number) + 1
        
        separator = '/'
        new_version_object_key = separator.join(object_key.split('/')[:-2]) + separator + str(self.version_number) + separator + object_key.split('/')[-1]
        print('New object key = ' + new_version_object_key)
    
s3_version_obj = S3CombineVersioning()
s3_version_obj.update_current_version("bucket_name/folder_name/2/object_name.py")
```
Output:    
```
Old Version Number = 2
New object key = bucket_name/folder_name/3/object_name.py
```
There can be multiple methods to add versioning object wise. These are my methods that I have used. 

## 6. Summary

In this blog we have covered what AWS S3 service is and its advantages. AWS S3 being a great storage service provider, we still face trouble in Object specific versioning. So we have found out our own solution with two different cases. One where every object is following their own version and later one is when more than one object is following the same version number. 

That’s it for now . If you have something else that is not covered in this then let me know in the comments below. 
Happy Reading!! 

