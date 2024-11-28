<center><img src="{{ site.baseurl }}/tutheaderbl.png" alt="Img"></center>

To add images, replace `tutheaderbl1.png` with the file name of any image you upload to your GitHub repository.

## Biodiversity Hotspot Mapping

### Tutorial Aims

##### 1. Understand the variety of functions in the `dplyr` package and its application to biodiversity data.

##### 2. Learn to creatively combine and manipulate biodiversity tables and spatial data.

##### 3. Become efficient and creative in manipulating species occurrence data and environmental variables.


### Tutorial Steps

##### <a href="#section1">Introduction</a>
- <a href="#section2">Prerequisites</a>
- <a href="#section3">Data and Materials</a>

##### <a href="#section2">PART I: Data Inspection and Cleaning</a>

##### <a href="#section3">PART II: Geospatial Data Analysis</a>
- <a href="#section4">Visualize Species Occurrence</a>
- <a href="#section5">Perform Kernel Density Estimation (KDE)</a>
- <a href="#section6">Moran's I for Spatial Autocorrelation</a>



---------------------------
### <a name="section1">Introduction</a>

Biodiversity hotspot mapping is the process of identifying areas on Earth that are particularly rich in species diversity and are under threat from human activities. These hotspots are often characterized by high levels of unique species found nowhere else, making them critical for conservation efforts. Mapping these areas allows us to prioritize regions for protection and sustainable management.

In this tutorial, we focus on how to map biodiversity hotspots using species occurrence data and environmental factors. We start by retrieving data on species, such as the gorilla, from sources like the [Global Biodiversity Information Facility (GBIF)](https://www.gbif.org/zh/), which provides information on where different species are found.

By visualizing the occurrence of species over environmental factors, we can identify areas where species thrive. This can help us understand the relationship between species distribution and the environment. You'll learn how to handle and visualize biodiversity data in `R` using the `dplyr` package for data manipulation, and spatial analysis techniques with `ggplot2` and `raster` to create informative maps.

The goal of biodiversity hotspot mapping is not just to show where species live, but to inform conservation strategies by focusing attention on areas that need protection the most. This process ultimately helps in preserving our planet’s most vulnerable ecosystems and species.


<p align="center">
  <div style="display:inline-block; text-align:center; width:45%; margin-right: 10px;">
    <img src="figures/worldmap_biological_hotspot.png" width="100%" />
    <br>
    <i style="font-size: 0.9em; text-align: center; display: block;">
      Map showing global biodiversity hotspots (Weller et al. 2014).<br>
      (Source: <a href="https://atlas-for-the-end-of-the-world.com/world_maps/world_maps_biological_hotspots.html" style="color: inherit; text-decoration: none;">ATLAS for the END of the WORLD</a>)
    </i>
  </div>
</p>


#### <a name="section2">Prerequisites</a>

This tutorial is suitable for those with a basic understanding of data manipulation and visualization in `R`. We will cover concepts ranging from basic data cleaning and transformation with `dplyr`, to more advanced spatial analysis with `ggplot2` and `raster`. To make the most of this tutorial, you should be familiar with:

- Data manipulation using `dplyr` and `tidyr`
- Data visualization with `ggplot2`

If you're new to these topics or need a refresher, there are excellent resources available on the Coding Club website:

[Getting started with R and RStudio](https://ourcodingclub.github.io/tutorials/intro-to-r/)

[Basic data manipulation](https://ourcodingclub.github.io/tutorials/data-manip-intro/)

[Beautiful and informative data visualisation](https://ourcodingclub.github.io/tutorials/datavis/)


#### <a name="section3">Data and Materials</a>

You can find all the data that you require for completing this tutorial on this [GitHub repository](https://github.com/EdDataScienceEES/tutorial-xiongshizhao). We encourage you to download the data to your computer and work through the examples throughout the tutorial as this will reinforces your understanding of the concepts taught in the tutorial.

**Now, let's get started!**


#### <a href="#section2">PART I: Data Inspection and Cleaning</a>

Begin by setting up a new R script. At the very top, include a few lines to introduce the project, such as your name, the date, and a brief description. When going through the tutorial, copy the individual code chunks and paste them to your script. Use hash symbol `#` when adding comments.

```r
# Biodiversity Hotspot
# Author: Your Name
# Date: DD/MM/YYYY
```

Setting the working directory, install and load libraries of all packages required for this tutorial.

```r
# Set working directory to where you saved the folder with tutorial materials on your computer
setwd("your_filepath")

# Load necessary libraries
# Uncomment the following lines to install the required packages if not already installed
# install.packages("dplyr")
# install.packages("ggplot2")

# Load required libraries
install.packages("dplyr")
install.packages("ggplot2")

# Load the gorilla occurrence data
gorilla <- read.csv("data/gorilla.csv")
```

Before start our work like visual or even our final goal-hotspot, we need to inspect the dataset to understand its structure and identify potential issues.

```r
# Inspect the dataset to understand its structure and identify potential issues
summary(gorilla)
```

After inspecting the brief features of our data, we found that there are NAs, but we don't want to remove all of them. If we do, there may be very little data left, so we will only focus on removing the NAs that may affect our subsequent graphing and mapping.

Next, let's handle missing data, and ensure no empty species entries.

```r
# Remove rows with empty species names and check for missing data
gorilla_data_clean <- gorilla_data %>%
  filter(species != "") %>%
  filter(!is.na(species), !is.na(decimalLatitude), !is.na(decimalLongitude))

# Rename the column 'decimalLongitude' to 'longitude' and 'decimalLatitude' to 'latitude'
gorilla_data_clean <- gorilla_data_clean %>%
  rename(longitude = decimalLongitude, latitude = decimalLatitude)

# Verify the updated column names
colnames(gorilla_data_clean)
```

Remember, the technique of combining two separate blocks of code into one involves chaining operations together with the pipe operator `%>%`, a key feature of the `dplyr` package in `R`.

We can apply this to reduce redundancy. Try the code below—it looks better, right? You can keep the following code and delete the one previously used for data cleaning.

```r
# Clean the dataset
gorilla_clean <- gorilla %>%
  # Remove rows with empty or missing species names and coordinates
  filter(!is.na(species) & species != "") %>%
  filter(!is.na(decimalLatitude) & !is.na(decimalLongitude)) %>%
  # Rename columns for consistency and simplicity
  rename(
    longitude = decimalLongitude,
    latitude = decimalLatitude
  )

# Validate the cleaned data
# Check for duplicate rows
gorilla_clean <- gorilla_clean %>%
  distinct()

# Summarize the cleaned dataset to verify the cleaning process
glimpse(gorilla_clean)
summary(gorilla_clean)

# Check the first few rows of the cleaned data for a quick inspection
head(gorilla_clean)
```

You may also want to save the cleaned data for future use.
```r
# Optional
write.csv(gorilla_clean, "data/gorilla_clean.csv")
```

Notice that we used the `glimpse` function above. We can see that although we focus on gorillas, there are more specific species under the gorilla genus: ***Gorilla gorilla*** and ***Gorilla beringei***. We will use an r command to help us print out this result manually.

```r
# Get the unique species
unique_species_list <- gorilla_clean %>%
  distinct(species) %>%  # Select unique species from the dataset
  pull(species)          # Extract the unique species as a vector

# Display the number of unique species in the dataset
cat("Number of unique species in the dataset:", length(unique_species_list), "\n")

# Display the list of unique species
cat("Unique species in the dataset:\n")
print(unique_species_list)
```


### <a name="#section3">PART II: Geospatial Data Analysis</a>

#### <a name="#section4">Visualize Species Occurrence</a>

This section focuses on visualizing the occurrence of two gorilla species, ***Gorilla gorilla (Western Gorilla)*** and ***Gorilla beringei (Eastern Gorilla)***, based on their geographic coordinates (longitude and latitude).

```r
# Plotting the occurrences of both species
plot<- ggplot(gorilla_clean, aes(x = longitude, y = latitude, color = species)) +
  geom_point(alpha = 0.6) +
  ggtitle("Gorilla Occurrences (Gorilla gorilla and Gorilla beringei)")
```

Remember what we learn from [Beautiful and informative data visualisation](https://ourcodingclub.github.io/tutorials/datavis/), You may want to add more details to make your visual even more beautiful!

Below is my graph. Show me yours! Don't forget to install the new packages for graphing.

```r
# Install packages
install.packages("RColorBrewer")
install.packages("extrafont")

# Load additional libraries
library(RColorBrewer)
library(extrafont)

# Create a ggplot for visualizing the gorilla occurrence data
beautifulplot<- ggplot(gorilla_clean, aes(x = longitude, y = latitude, color = species)) +
  geom_point(alpha = 0.7, size = 3, shape = 21, stroke = 0.8) +  # Slightly larger, more defined points
  scale_color_manual(values = c("Gorilla gorilla" = "#FF9A8B",
                                "Gorilla beringei" = "#6A9ECF")) +
  labs(
    title = "Gorilla Occurrences\n(Gorilla gorilla and Gorilla beringei)",  # Title
    x = "Longitude",
    y = "Latitude",
    color = "Species"
  ) +
  theme_minimal(base_size = 16) +  # Base size for readability
  theme(
    plot.title = element_text(family = "Georgia", hjust = 0.5, size = 22, color = "darkslateblue", face = "bold"),  # Fancy script title with Lobster font
    axis.title = element_text(family = "Georgia", size = 16, face = "italic", color = "darkred"),  # Elegant font for axis titles
    axis.text = element_text(size = 12, family = "Georgia", color = "black"),  # Readable axis text
    legend.position = "right",  # Legend on the right
    legend.title = element_text(size = 15, family = "Georgia", face = "bold"),  # Bold legend title
    legend.text = element_text(size = 13, family = "Georgia"),  # Font size for legend
    panel.grid.major = element_line(color = "gray90", size = 0.5),  # Light major grid lines
    panel.grid.minor = element_line(color = "gray95", size = 0.25),  # Even lighter minor grid lines
    panel.border = element_rect(color = "black", fill = NA, size = 1.2),  # Plot border
    plot.margin = margin(50, 50, 50, 50),  # Balance the margins
    plot.background = element_rect(fill = "aliceblue", color = NA)  # Soft background color
  )
```

Whatever you create from the data, don't forget to make an 'outputs' folder to save them. Make it a good habit for organizing well.

```r
# Save the plots
ggsave("outputs/gorilla_plot.png", plot = plot, width = 6, height = 4, dpi = 300)
ggsave("outputs/gorilla_beautifulplot.png", plot = beautifulplot, width = 6, height = 4, dpi = 300)
```







At the beginning of your tutorial you can ask people to open `RStudio`, create a new script by clicking on `File/ New File/ R Script` set the working directory and load some packages, for example `ggplot2` and `dplyr`. You can surround package names, functions, actions ("File/ New...") and small chunks of code with backticks, which defines them as inline code blocks and makes them stand out among the text, e.g. `ggplot2`.

When you have a larger chunk of code, you can paste the whole code in the `Markdown` document and add three backticks on the line before the code chunks starts and on the line after the code chunks ends. After the three backticks that go before your code chunk starts, you can specify in which language the code is written, in our case `R`.

To find the backticks on your keyboard, look towards the top left corner on a Windows computer, perhaps just above `Tab` and before the number one key. On a Mac, look around the left `Shift` key. You can also just copy the backticks from below.

```r
# Set the working directory
setwd("your_filepath")

# Load packages
library(ggplot2)
library(dplyr)
```

<a name="section2"></a>

## 2. The second section

You can add more text and code, e.g.

```r
# Create fake data
x_dat <- rnorm(n = 100, mean = 5, sd = 2)  # x data
y_dat <- rnorm(n = 100, mean = 10, sd = 0.2)  # y data
xy <- data.frame(x_dat, y_dat)  # combine into data frame
```

Here you can add some more text if you wish.

```r
xy_fil <- xy %>%  # Create object with the contents of `xy`
	filter(x_dat < 7.5)  # Keep rows where `x_dat` is less than 7.5
```

And finally, plot the data:

```r
ggplot(data = xy_fil, aes(x = x_dat, y = y_dat)) +  # Select the data to use
	geom_point() +  # Draw scatter points
	geom_smooth(method = "loess")  # Draw a loess curve
```

At this point it would be a good idea to include an image of what the plot is meant to look like so students can check they've done it right. Replace `IMAGE_NAME.png` with your own image file:

<center> <img src="{{ site.baseurl }}/IMAGE_NAME.png" alt="Img" style="width: 800px;"/> </center>

<a name="section1"></a>

## 3. The third section

More text, code and images.

This is the end of the tutorial. Summarise what the student has learned, possibly even with a list of learning outcomes. In this tutorial we learned:

##### - how to generate fake bivariate data
##### - how to create a scatterplot in ggplot2
##### - some of the different plot methods in ggplot2

We can also provide some useful links, include a contact form and a way to send feedback.

For more on `ggplot2`, read the official <a href="https://www.rstudio.com/wp-content/uploads/2015/03/ggplot2-cheatsheet.pdf" target="_blank">ggplot2 cheatsheet</a>.

Everything below this is footer material - text and links that appears at the end of all of your tutorials.

<hr>
<hr>

#### Check out our <a href="https://ourcodingclub.github.io/links/" target="_blank">Useful links</a> page where you can find loads of guides and cheatsheets.

#### If you have any questions about completing this tutorial, please contact us on ourcodingclub@gmail.com

#### <a href="INSERT_SURVEY_LINK" target="_blank">We would love to hear your feedback on the tutorial, whether you did it in the classroom or online!</a>

<ul class="social-icons">
	<li>
		<h3>
			<a href="https://twitter.com/our_codingclub" target="_blank">&nbsp;Follow our coding adventures on Twitter! <i class="fa fa-twitter"></i></a>
		</h3>
	</li>
</ul>

### &nbsp;&nbsp;Subscribe to our mailing list:
<div class="container">
	<div class="block">
        <!-- subscribe form start -->
		<div class="form-group">
			<form action="https://getsimpleform.com/messages?form_api_token=de1ba2f2f947822946fb6e835437ec78" method="post">
			<div class="form-group">
				<input type='text' class="form-control" name='Email' placeholder="Email" required/>
			</div>
			<div>
                        	<button class="btn btn-default" type='submit'>Subscribe</button>
                    	</div>
                	</form>
		</div>
	</div>
</div>