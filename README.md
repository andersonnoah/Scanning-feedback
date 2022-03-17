### This is feedback *my* experience.
I acknowledge that this documentation is written for an audience of developers already doing work on the middleware app.

**Steps that need completed before beginning working on adding new scanned photograpy:**
* Share necessary credentials via LastPass
	* Note: Create a LastPass account before credentials are shared to avoid having to share credentials twice
* Add whoever is working on this as a collaborator on ClevelandMuseumArt GitHub account

**Things still left unanswered:**
* How to interface/general information on `artlens-cms`
* How to interface/general information on `artlens.clevelandart.org`
	* Are these the same server?

**Other ideas:**
* If one person is tasked with making artworks scannable, it might be worth it to have one larger document that includes the steps to create hotspot content in the middleware.
* While it might be too tedious for what it's worth in markdown, embedded images alongside documentation is always helpful.

# Adding New Scanned Photography

### Preparing Images From In-Gallery Photography

_(This assumes you have multiple images for each artwork)_

* Copy all your new artwork photography images to an accession-number-name directory, e.g. `/my/new/image/location/for/artwork/1974.16`.

* Crop images **~~in photoshop~~** so that you are capturing minimal amount of background. The exception to this is that it is helpful to include the frame of paintings (as long as it's the currently installed frame). Save as "high" (not "maximum") quality JPEGs.

* Rename and resize images using the `rename_and_resize_images.py` script.
	* First param is file location (required). This directory path MUST end in the accession number of the artwork.
	* Second param is starting index (optional, default 1) for renaming files. If it's a new artwork, use 1, else find the last indexed image from the full_museum.xml in vuforia data.
	* Third param is bounding box for resizing (optional, default 1920,1440). Use 1920x1440 or 1440x1920, depending on oriention.

* EXAMPLE: `python rename_and_resize_images.py /Users/ethanholda/Downloads/scanning/1974.16 1440,1920`

### Preparing Images From Piction Collection Images

_(This assumes you use studio photography for the trackable)_

* If the artworks are prints, paintings, etc, you can download multiple artworks from Piction by running:  
`python get_images_from_piction.py <download directory> <comma-delimited accession numbers>`.

* Rename/resize these by running:  
`python rename_resize_all.py <download directory>`.


### Adding Images to Vuforia DB and App

**NOTE:** Vuforia won't accept any image bigger than 2MB.*
	* *Whether rename_resize_all.py takes care of this or not, in my opinion, this should be noted **before** you start editing and resizing images, not after.*

* Log into the [Vuforia developer portal](https://developer.vuforia.com/).
	* ***Having never used Vuforia before, I had slight difficulty locating "Target Manager". Not suggesting any changes, just documenting my experience.***

* Go to the "Target Manager" and select the "full_museum" database.

* Add targets. For "width" just put 100.

	***Note: Here, I think it would have been helpful to know that it may take some time for images to process. I ended up jumping ahead to downloading the "full_museum" database before processing finished, resulting in needing to re-download the entire database once processing was completed.***

	***Additionally, knowing that Vuforia automatically scores the quality of the targets uploaded out of five stars would have been useful to know. While one of my images scored a 3/5 stars and scanned well, you might consider setting a lower limit scoring rule (e.g., if the target score is lower than 3/5 stars, retake or edit the photo(s) until target score is at least a 3/5). Ensuring a quality image beforehand saves you the trouble of needing to re-do steps.***

* Prepare bundle:
  * Download the "full_museum" database from target manager.
  * Rename `full_museum.zip` to `vuforia_data.zip`.
	  * ***It might be clearer to say: Rename "full_museum".zip to "vuforia_data".zip -- I could see someone changing the name to "vuforia_data.zip".zip*** -- *Yes, it is an allowed format. I tested it.*
  * Make a copy of the current live app data on S3 so you can roll back: `aws s3 cp s3://cma-artlens/v2 s3://cma-artlens/v2_<TIMESTAMP> --recursive`
	  * ***I would format the command so it's just a simple copy/paste. To achieve your desired TIMESTAMP format of "YYYYMMDD-hms" you could edit the command as follows:***
  ```aws s3 cp s3://cma-artlens/v2 s3://cma-artlens/v2_$(date +"%y%m%d-%H%M%S") --recursive```

  * Put `vuforia_data.zip` in the `/mnt/data/bundles` file on artlens-cms.
	  * ***Understanding that a developer working on the middleware would know how to interface with 'artlens-cms', to anyone else it might not be clear. The same is true for completeing the step below.***
  * Run the `python manage.py update_manifest -u v2` flask script on artlens-cms.
	  * ***While abstracting what `manage.py` does is probably fine, it might be useful to give a brief statement explaining what it does. I acknowledge that the function of `manage.py` file could probably be deciphered when taking into account "update_manifest" and "flask script".***

* **Place all images in an accession-numbered folder in this repo, e.g. `./photography/1974.16`, commit to master branch, and push.**
	* While *I think* extended descriptions have an unspoken format, for consistency it might be a nice addition to ask for the inclusion of at least the accession numbers (e.g., "Added `<acc nbr>` and `<acc nbr>`)."
