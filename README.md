# docker-publish-images
Script for publishing images to DockerHub

``` sh
#!/bin/bash

git checkout master
git pull origin master
rm -Rf vendor
cps install --no-dev
for branch in $(git branch --list | grep -Eo "^\s*docker-image/(.*)"); do
    VERSION=$(echo "$branch" | cut -d/ -f2)
    CONTAINER_NAME="apisearch/server/$VERSION"
    IMAGE_NAME="apisearchio/search-server:$VERSION"
    git checkout $branch
    echo "Pushing $VERSION to Docker hub"
    git rebase master
    git log master..$branch
    git diff master..$branch
    rm -Rf Tests
    read -p "Ready to push (y/n)?" choice
    case "$choice" in
        y|Y ) echo "Let's push it";;
        n|N ) exit;;
        * ) echo "invalid";;
    esac
    git push --force origin $branch
    docker build -t "$CONTAINER_NAME" .
    docker tag $CONTAINER_NAME:latest $IMAGE_NAME
    docker push $IMAGE_NAME
    git checkout -- Tests
done
git checkout master
git reset --hard HEAD
cps install --dev
```
