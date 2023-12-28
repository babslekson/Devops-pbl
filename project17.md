# AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 2
---
## Setup Networking of the Architecture
---
```bash
tags = {
    Owner-Email = "olalekanbabayemi1@gmail.com"
    Managed-By = "terraform"
}
```
- Create 4 private subnets
```bash
resource "aws_subnet" "private" {
    count                      = var.preferred_number_of_private_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_private_subnets
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = cidrsubnet(var.vpc_cidr, 8, count.index + 2)
    availability_zone          = data.aws_availability_zones.available.names[count.index]


    tags = merge (
        var.tags, 
        {
            Name = format("%s-private-subnet-%s", var.name , count.index +  1)
        },
    )
}
```
- Create `internet_gateway.tf`file
- Create Internet Gateway
```bash
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id

   tags = merge( var.tags,
    {
    Name = format ("%s-IGW", var.name )
    },
  )
}
```
- Create `nat_gateway.tf`file
- Create one NAT Gateway, create Elastic IP and attach it to NAT gateway

```bash
resource "aws_eip" "nat-eip" {
    
     depends_on = [aws_internet_gateway.igw]
}
resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat-eip.id
  subnet_id     = element(aws_subnet.public.*.id, 0 )
    depends_on = [aws_internet_gateway.igw]

  tags = merge( var.tags,
    {
    Name = format ("%s-NAT", var.name )
    },
  )
}
```
Create `route_tables.tf` file
Create private route table
```bash
resource "aws_route_table" "private-rtb" {
  vpc_id = aws_vpc.main.id

 tags = merge( var.tags,
    {
    Name = format ("%s-private-rtb", var.name )
    },
  )
}
```
- Create route for the private route table and attach a nat gateway to it 
```bash
resource "aws_route" "private-rt" {
  route_table_id            = aws_route_table.private-rtb.id
  destination_cidr_block    = "0.0.0.0/0"
  nat_gateway_id          = aws_nat_gateway.nat.id
  ```
  - Associate all private subnets to private route table
  ```bash
  resource "aws_route_table_association" "private-route-association" {
  count = length ( aws_subnet.private[*].id)
  subnet_id      = element (aws_subnet.private[*].id , count.index)
  route_table_id = aws_route_table.private-rtb.id
  }
 ```
 - Create public route table
 ```bash
 resource "aws_route_table" "public-rtb" {
  vpc_id = aws_vpc.main.id

 tags = merge( var.tags,
    {
    Name = format ("%s-public-rtb", var.name )
    },
  )
}
 ```
- Create route for the public route table and attach a internet gateway to it 
```bash
resource "aws_route" "public-rt" {
  route_table_id            = aws_route_table.public-rtb.id
  destination_cidr_block    = "0.0.0.0/0"
  gateway_id                = aws_internet_gateway.igw.id
}

```
 - Associate all public subnets to public route table
```bash
resource "aws_route_table_association" "public-route-association" {
  count = length ( aws_subnet.public[*].id)
  subnet_id      = element (aws_subnet.public[*].id , count.index)
  route_table_id = aws_route_table.public-rtb.id
}
```
