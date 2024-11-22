## About

`dcm_qa_philips_dwi` is a simple DICOM to NIfTI validator script and dataset. This repository is similar to [dcm_qa](https://github.com/neurolabusc/dcm_qa), but includes data from Philips MRI scanners. Specifically, this dataset demonstrates volume ordering for [Diffusion Weighted Imaging (DWI)](https://github.com/rordenlab/dcm2niix/issues/546).

The image in this repsoitory were exported as both enhanced and classic DICOMs.

## Details

Data provided by [Paul S. Morgan at the University of Nottigham](https://www.nottingham.ac.uk/research/beacons-of-excellence/precision-imaging/our-experts/paul-morgan/index.aspx). Data was saved as both classic (where each slice is a separate file) as well as enhanced DICOM (where an entire series of images is saved as a single file) format. 

* `701_DTI_Biobank_2mm_MB3S2_EPI`
  * Manufacturer: Philips
  * Model: Ingenia Elition X
  * Implementation Version: Philips MR 57.0
  * Software Versions: 5.7.1.2
  * 12 volumes with b=1000 s/mm2, each with a different direction
  * 5 volumes with virtually no b-weighting (b= 0, 0.001, 0.002, 0.003, 0.004) and same `direction`

## Ordering of Volumes
 
The DICOM standard does not require that the image instance number (0020,0013) is sequential or even unique. However, with the exception of Philips manufacturers generate instance numbers in the temporal order of acquisition. Unfortunately, Philips often generates scrambled instance numbers. 

For Philips software 5.6 and later one can use the private DiffusionOrder tag (2005,1596) to determine which 3D volume 2D slice belongs to. This tag reveals the temporal order of acquisition. When available, dcm2niix will use this tag to sort diffusion images.

The situation is trickier for older Philips diffusion data, where temporal order can not be precisely inferred. As demonstrated in this dataset, Philips can use the same FrameAcquisitionDateTime ([0018,9074](http://dicomlookup.com/lookup.asp?sw=Tnumber&q=(0018,9074))) for all images in a series. This makes it impossible to infer the true acquisition order of volumes for Philips DWI images. For enhanced DICOMs, dcm2niix will order volumes as described by the DimensionIndexValues ([0020,9157](http://dicomlookup.com/lookup.asp?sw=Tnumber&q=(0020,9157))). However, this tag does not exist for classic DICOM. Therefore, for Philips DICOM saved as classic DICOMs, dcm2niix will resort to using the private tags BValueNumber (2005,1412) and GradientOrientationNumber (2005,1413) to identify slices from the same volume. As this dataset illustrates, it is possible for volumes in a series to repeat both of these tags, but taken together they provide a unique combination. For ordering Philips classic DWI volumes, dcm2niix will sort by 2005,1412 and then by 2005,1413. Therefore, volumes with identical b-values will be stored contiguously. Be aware that this volume order may not match temporal order of acquisition, or the order specified if the data is saved as enhanced format. As a concrete example, here is the volume order for the example dataset (where 2001,1003 is the specified b-value in s/mm2). Therefore, when converting older Philips images, users should be cautious if they use tools that infer temporal order (e.g. for modeling head motion or T1 effects).

Here are the relevant tags for the provided dataset.

| 2005,1412 | 2005,1413 | 2001,1003 | 2005,1596 |
|-----------|-----------|-----------|-----------|
| 1 | 1 | 0 | 1 |
| 2 | 2 | 1000 | 2 |
| 2 | 3 | 1000 | 3 |
| 2 | 4 | 1000 | 4 |
| 2 | 5 | 1000 | 6 |
| 2 | 6 | 1000 | 7 |
| 2 | 7 | 1000 | 8 |
| 2 | 8 | 1000 | 10 |
| 2 | 9 | 1000 | 11 |
| 2 | 10 | 1000 | 12 |
| 2 | 11 | 1000 | 14 |
| 2 | 12 | 1000 | 15 |
| 2 | 13 | 1000 | 16 |
| 3 | 1 | 0.001 | 5 |
| 4 | 1 | 0.002 | 9 |
| 5 | 1 | 0.003 | 13 |
| 6 | 1 | 0.004 | 17 |

## License

The code and images are covered by the [2-clause BSD license](https://opensource.org/licenses/BSD-2-Clause).

## Running

Assuming that the executable dcm2niix is in your path, you should be able to simply run the script `batch.sh` from the terminal.

If you have problems you can edit the first few lines of the `batch.sh` script so that `basedir` reports the explicit location of the `dcm_qa` folder (by default this is assumed to be the folder containing the script) on your computer and `exenam` reports the explicit location of dcm2niix (by default it is assumed to be in your path). Also, make sure the script is executable (`chmod +x batch.sh`). Then run the script.

## Useful Links

 - dcm2niix's handling of these files is described [here](https://github.com/rordenlab/dcm2niix/tree/master/Philips).

