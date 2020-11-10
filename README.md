# slim-data-release-template


## code

Need to have non-CRAN packages`dssecrets`, `meddle`, and `scipiper` installed, among other packages. 
Need to have CRAN package `sbtools` installed

- Create a data release or create a child item on sciencebase to experiment on
- Add "write" permissions on the release item for `cidamanager` (this is the `dssecrets` service account)
- Change the files and the functions in `src/` to what you need
- Edit data release information in `in_text/text_data_release.yml` to fit your data release and your file names and contents
- modify the sciencebase indentifier to your parent data release identifier (should be a string that is something like "5faaac68d34eb413d5df1f22")
- run `scmake()`
- validate your `out_xml/fgdc_metadata.xml` file with the [validator tool](https://mrdata.usgs.gov/validation/)
- fix any validation errors (usually this requires filling in metadata information in the `in_text/text_data_release.yml` and perhaps looking a the [metadata template](https://raw.githubusercontent.com/USGS-R/meddle/master/inst/extdata/FGDC_template.mustache))
- win

## remake.yml details

This slim template is designed to keep everything in a single remake yaml. So all data munging, manipulation, and file writing happens there, in addition to the sciencebase uploads

```yaml
targets:
  all:
    depends:
      - sb_xml
      - sb_data
    
```
These are the two different "pushes" to sciencebase, one for xml (metadata) and another for data. It is somewhat arbitrary how these are split up (there could be a single push for all files, or a push for each individual file). I do it this way because I want metadata edits separate from data files, so that the data files aren't replaced everytime I fix a metadata typo or add information to a metadata field. 

```yaml
  sbid:
    command: c(I('5faaac68d34eb413d5df1f22'))
```
Here we define the data release location on sciencebase using the sciencebase identifier. 

```yaml
  sf_spatial_data:
    command: st_read('example_data/example_shapefile/example_shapefile.shp')
    depends:
      - example_data/example_shapefile/example_shapefile.dbf
      - example_data/example_shapefile/example_shapefile.prj
      - example_data/example_shapefile/example_shapefile.shx
  
  spatial_metadata:
    command: extract_feature(sf_spatial_data)
```
Read in (`st_read`) and manipulate data here. `extract_feature()` is from the `meddle` package and builds a list of structured spatial information for use in metadata files.

```yaml
  out_data/cars.csv:
    command: file.copy(from = "example_data/example_cars.csv", 
      to = target_name)
  

  out_data/spatial.zip:
    command: sf_to_zip(zip_filename = target_name, 
      sf_object = sf_spatial_data, layer_name = I('spatial_data'))
```
write final metadata files as you want them to appear in the data release. 

```yaml
  sb_data:
    command: sb_replace_files(sbid,
      "out_data/spatial.zip",
      "out_data/cars.csv")
      
  sb_xml:
    command: sb_render_post_xml(sbid,
      "in_text/text_data_release.yml",
      spatial_metadata, 
      xml_file = I("out_xml/fgdc_metadata.xml"))
```
Push the files to sciencebase using these two utility functions that are included in `src/sb_utils.R` of this repo template

