# get-latest-image-per-ecr-repo.sh

```bash
#!/bin/bash -
#===============================================================================
#
#          FILE: get-latest-image-per-ecr-repo.sh
#
#         USAGE: ./get-latest-image-per-ecr-repo.sh aws-account-id
#
#       AUTHOR: Enri Peters (EP)
#       CREATED: 04/07/2022 12:59:15
#=======================================================================

set -o nounset       # Treat unset variables as an error

for repo in \
        $(aws ecr describe-repositories |\
        jq -r '.repositories[].repositoryArn' |\
        sort -u |\
        awk -F ":" '{print $6}' |\
        sed 's/repository\///')
do
        echo "$1.dkr.ecr.eu-west-1.amazonaws.com/${repo}@$(aws ecr describe-images\
        --repository-name ${repo}\
        --query 'sort_by(imageDetails,& imagePushedAt)[-1].imageDigest' |\
        tr -d '"')"
done > latest-image-per-ecr-repo-${1}.list
```

The output will be written to a file named latest-image-per-ecr-repo-awsaccountid.list.
An example of this output could be:
```
123456789123.dkr.ecr.eu-west-1.amazonaws.com/your-ecr-repository-name@sha256:fb839e843b5ea1081f4bdc5e2d493bee8cf8700458ffacc67c9a1e2130a6772a
...
...
```
With this you can do something like below to pull all the images to your machine.
```bash
#!/bin/bash -

for image in $(cat latest-image-per-ecr-repo-353131512553.list)
do
    docker pull $image
done
```

You will see that when you run docker images that none of the images are tagged. But you can 'fix' this by running these commands:
`docker images --format "docker image tag {{.ID}} {{.Repository}}:latest" > tag-images.sh`
`chmod +x tag-images.sh`
`./tag-images.sh`

Then they will all be tagged with latest on your machine.
