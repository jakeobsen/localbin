#!/usr/bin/env python3
# NASA Astronomy Picture of the Day downloader
# Copyright (C) 2020 Morten Jakobsen
#
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation; either version 2
# of the License, or (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301, USA.
# Or use this link: https://www.gnu.org/licenses/old-licenses/gpl-2.0.txt

import hashlib
import os
from datetime import datetime
from os import path
import argparse
import requests

# Get arguments
parser = argparse.ArgumentParser(description='Download NASA Astronomy Picture of the Day')
parser.add_argument('output_dir', type=str, nargs='?', default="~/Picture/NASA/",
                    help='Output dir for download script.')
parser.add_argument('--url', '-u', type=str, nargs='?', default="https://apod.nasa.gov/apod/",
                    help='Custom URL to download from')
args = parser.parse_args()

# Download image from NASA's APOD
base_url = "https://apod.nasa.gov/apod/"

# Destination directory where files are stored
destination_dir = path.expanduser(args.output_dir)

# Location of the latest file (the last image is symlinked here at the end)
latest_file = path.join(destination_dir, "latest")

# Download APOD webpage
r = requests.get(args.url)
if r.status_code == 404:
    print(f"Page not found: {args.url}")

date = None

# Loop over the file and do things
for line in r.text.split("\n"):

    # Lets try and parse the date that sometimes appears in the title tag
    if "<title>" in line.lower():
        try:
            date = line.split(':')[1]
            date = date.split('-')[0]
            date = datetime.strptime(date.strip(), "%Y %B %d")
        except Exception as e:
            date = datetime.now()

    # Download the image
    if "image/" in line.lower() and "href" in line.lower():

        # Set date if it's none
        if date is None:
            date = datetime.now()

        # Fetch the image url and create image link
        link = line.split('"')[1]
        image_link = f"{base_url}{link}"

        # Output file
        destination_file = os.path.join(
            destination_dir,
            "_".join([
                date.strftime('%Y/%m/%Y-%m-%d'),
                path.basename(image_link)
            ])
        )

        # Create directory if it does not exists
        if not path.isdir(path.dirname(destination_file)):
            os.makedirs(path.dirname(destination_file), exist_ok=True)

        # No need to do more work if file exists
        if not path.isfile(destination_file):

            # Grab the MD5 of the latest file
            if path.islink(latest_file):
                try:
                    with open(latest_file, "rb") as f:
                        last_image = f.read()
                    last_md5 = hashlib.md5(last_image).hexdigest()
                except FileNotFoundError as e:
                    last_md5 = None
            else:
                last_md5 = None

            # Download new image and generate MD5
            image = requests.get(image_link, allow_redirects=True)
            image_md5 = hashlib.md5(image.content).hexdigest()

            # Since NASA timezone and my timezone is not aligned and the file from yesterday might still
            # be the "latest" file on NASA's website, then we check if the current file's MD5 sum matches
            # what we downloaded. If it does, then we don't do anything as we already have the latest file.
            if image_md5 != last_md5:

                # Save downloaded image
                with open(destination_file, "wb") as f:
                    f.write(image.content)

                # Unlink last image
                if path.islink(latest_file):
                    os.unlink(latest_file)

                # Relink downloaded image
                if not path.islink(latest_file):
                    os.symlink(destination_file, latest_file)

                print(f"Downloaded image: {destination_file}")

        # End of execution - break out of for loop
        break
