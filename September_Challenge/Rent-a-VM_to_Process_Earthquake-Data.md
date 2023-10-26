

## Task 2. SSH into the instance


```bash
	sudo apt-get update
	
	sudo apt-get -y -qq install git
	
	sudo apt-get install python-mpltoolkits.basemap
	
	git --version
	
```

## Task 4. Ingest USGS data

```bash
	git clone https://github.com/GoogleCloudPlatform/training-data-analyst
	cd training-data-analyst/CPB100/lab2b
	
	less ingest.sh
	
	bash ingest.sh
```


## Task 5. Transform the data

```bash
	## Install req py library
	bash install_missing.sh
	
	
	# Run
	python3 transform.py
```

## Task 6. Create a Cloud Storage bucket and copy files inside it


`` gsutil cp earthquakes.* gs://<YOUR-BUCKET>/earthquakes/ ``


------------------------------------------------------------------------------------END_GAME-------------------------------------------------------------
