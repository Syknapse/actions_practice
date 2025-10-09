# Practice in creating GitHub actions

Example repo where a podcast.xml file is automatically updated every time a new podcast entry is added to feed.yaml. 

On push the workflow in .github/workflows/main.yml is triggered.
This sets up the correct environment needed for running python, then runs the feed.py script.
This script takes the simple format of the yaml feed entry and turns it into the xml podcast entry. It then rewrites the podcast.xml file updating it and commiting the result. 

## Publishing a marketplace action

Create a repo for the action to be published, add readme and license

Add the script file that needs to be executed 

Create an entrypoint file

- Example entrypoint.sh
    
    ```bash
    #!/bin/bash
    
    echo "========ENTRYPOINT START==========="
    
    git config --global user.name "${GITHUB_ACTOR}"
    git config --global --add safe.directory /github/workspace
    
    python3 /usr/bin/feed.py
    
    git add -A && git commit -m "Update feed"
    git push --set-upstream origin main
    
    echo "=========ENTRYPOINT END============"
    ```
    

Change entrypoint.sh write permissions making it executable
 `chmod -R 775 entrypoint.sh`

Create a Dockerfile 

- Example Dockerfile
    
    ```docker
    FROM ubuntu:latest
    
    RUN apt-get update && apt-get install -y \
    	python3.10 \
    	python3-pip \
    	git
    	
    RUN pip3 install PyYAML
    
    COPY feed.py /usr/bin//feed.py
    
    COPY entrypoint.sh /entypoint.sh
    
    ENTRYPOINT ["/entrypoint.sh"]
    ```
    

Create action file

- Example action.yml
    
    ```yaml
    name: "Podcast Generator"
    author: "Fulanito"
    description: "Generates a feed for a podcast from a YAML file"
    runs: 
    	using: "docker"
    	image: "Dockerfile"
    branding: 
    	icon: "git-branch" # from Feather icons
    	color: "red"
    inputs: 
    	email:
    		description: The commiter's email address
    		required: true
    		default: ${{ github.actor }}@localhost
    	name:
    		description: The commiter's name
    		required: true
    		default: ${{ github.actor }}
    ```
    

Now you can use this action from another repo

```yaml
name: Generate Podcast Feed
on: [push]
jobs: 
  build: 
    runs-on: ubuntu-latest
    steps:
     - name: Checkout Repo
       uses: actions/checkout@v3
     - name: Run feed generator # the action from the other repo
       uses: syknapse/custom-action@main # action repo: <gitHub_username>/<repo>@version/branch
```

To release action in GitHub Marketplace:

Draft a release (from side menu in repo or usually when GitHub detects it’s an action it gives you the option as a banner). Once you fill the details and publish the release it’ll be published in the market place.