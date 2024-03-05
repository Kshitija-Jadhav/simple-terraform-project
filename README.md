# simple-terraform-project

Pre-requisites
1. AWS account

Steps:
1. Launch and configure an EC2 instance.
2. Install Terraform using the below commands.
   
   sudo yum install -y yum-utils shadow-utils
   sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
   sudo yum -y install terraform
   
4. 3. Create a provider.tf file
    
    
    terraform {
      required_providers {
        aws = {
          source = "hashicorp/aws"
          version = "5.39.1"
        }
      }
    }
    
    provider "aws" {
      # Configuration options
        region = "us-east-2"
    }
    
    
4. Run terraform init
5. Now let’s create s3 resource. Remember to keep the name of the bucket unique.
    
    resource "aws_s3_bucket" "mybucket" {
      bucket = var.bucketname
    }
    
    
6. Now I will use the variables file to store the name of the bucket - variables.tf
    
   
    variable "bucketname"{
            default = "myterraforms3buck"
    }
   
    
7. If you see the bucket properties the static website option is disabled and in permissions, the public access is denied.
8. We will change the ownership of the bucket now
    
    
    resource "aws_s3_bucket" "mybucket" {
      bucket = var.bucketname
    }
    
    resource "aws_s3_bucket_ownership_controls" "example" {
      bucket = aws_s3_bucket.mybucket.id
    
      rule {
        object_ownership = "BucketOwnerPreferred"
      }
    }
  
    
9. Now let’s make the website public
    
    
    resource "aws_s3_bucket" "mybucket" {
      bucket = var.bucketname
    }
    
    resource "aws_s3_bucket_ownership_controls" "example" {
      bucket = aws_s3_bucket.mybucket.id
    
      rule {
        object_ownership = "BucketOwnerPreferred"
      }
    
    }
    
    resource "aws_s3_bucket_public_access_block" "example" {
      bucket = aws_s3_bucket.mybucket.id
    
      block_public_acls       = false
      block_public_policy     = false
      ignore_public_acls      = false
      restrict_public_buckets = false
    }
    
    
10. add ACL block in the main.tf - In Terraform, the Access Control List (ACL) block is used to configure the access control settings for an S3 bucket. S3 buckets support a feature called bucket ACLs, which define who can access the objects in the bucket and what permissions they have. The ACL block allows you to set permissions at the bucket level, controlling access to the entire bucket rather than individual objects. 
    
    
    resource "aws_s3_bucket_acl" "example" {
      depends_on = [
        aws_s3_bucket_ownership_controls.example,
        aws_s3_bucket_public_access_block.example,
      ]
    
      bucket = aws_s3_bucket.mybucket.id
      acl    = "public-read"
    }
  
    
11. Now we have enabled public access to our website. Let's now enable, static website hosting.
    
    For this, we will use website configuration resources.  Note: To use this resource file it is important to have an index.html file in your S3 bucket. So Let’s create the files first and then add those to the objects in the S3 bucket.
    
    For that, we will be using object resources.
    
    ```groovy
    resource "aws_s3_bucket_object" "index" {
      key                    = "index.html"
      bucket                 = aws_s3_bucket.mybucket.id
      source                 = "index.html"
    	acl                    = "public-read"
    	content_type           = "text/html"
    }
    
    resource "aws_s3_bucket_object" "error" {
      key                    = "error.html"
      bucket                 = aws_s3_bucket.mybucket.id
      source                 = "error.html"
    	acl                    = "public-read"
    	content_type           = "text/html"
    }
  
    
12. Now let’s configure the website
    
    
    resource "aws_s3_bucket_website_configuration" "website" {
      bucket = aws_s3_bucket.mybucket.id
    
      index_document {
        suffix = "index.html"
      }
    
      error_document {
        key = "error.html"
      }
    
      depends_on = [ aws_s3_bucket_acl.example ]
    }
    
13. Now our website is created, go to the S3 bucket and go to the link.
