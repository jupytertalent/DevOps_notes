buildspec.yml

''' 
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 20
    commands:
      - yum update -y
      - yum install nodejs -y
      - npm install

  build:
    commands:
      - npm run build 
      - aws s3 cp ./build s3://nodeapp-vignesh/ --recursive

 
artifacts:
  files:
    - 'build/*'

cache:
  paths:
    - node_modules  
'''