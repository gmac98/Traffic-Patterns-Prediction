# Traffic Patterns Prediction



# Summary

This project attempts to predict the AADT traffic volumes for each road within a road network, first utilizing a quadratic programming (QP)
optimizer, secondly by using neural networks.

This project was developed as the project for CS 6140 (Machine Learning), completed during the Spring 2021 semester at Northeastern
University.


# Approach

Unless specified, all commands will be assumed to run from this project's main directory.


## Data origin

This project utilizes map data obtained from OpenStreetMap, which is exported using the same XML format provided by OpenStreetMap.
AADT values must be provided as a list using the following JSON format (ID tag may be excluded):
```json
{
    "AADT":[
        {
            "ID":"a1",
            "latlon":[0, 0],
            "AADT μ":100,
            "AADT σ":20
        },
        {
            "ID":"b2",
            "latlon":[0, 1],
            "AADT μ":200,
            "AADT σ":40
        }
    ]
}
```

Where:
- μ: AADT mean
- σ: AADT standard deviation


## Quadratic Programming optimization

### Data pre-processing

The OpenStreetMap and AADT data must be preprocessed into a JSON output via:
```bash
python3 QP/preprocessing.py \
    --osm $OSM_FILE_FILEPATH \
    --aadt $AADT_JSON_FILEPATH \
    --output $OUTPUT_JSON_FILEPATH
```

For example:
```bash
python3 QP/preprocessing.py \
    --osm examples/Niles/Niles.osm \
    --aadt examples/Niles/Niles_AADT.json  \
    --output test/example1.json
```

Use the *--show* flag to show a map as well as node and road counts.


If requesting information about how the flags in more detail, run:
```bash
python3 QP/preprocessing.py --help
```


The following types of ways are not used when calculating traffic values:
* aeroway: Cannot drive (unless way is a highway)
* cycleway: Cannot drive (unless way is a highway)
* footway: Cannot drive (unless way is a highway)
* railway: Cannot drive (unless way is a highway)
* waterway: Cannot drive (unless way is a highway)
* highway/service/driveway: Not a road designed for continuous driving
* highway/service/parking_asile: Not a road designed for continuous driving

### Convex optimization:

Run the quadratic optimizer (OSQP solver) using the above JSON preprocessed output (note, the *--verbose* flag will have no effect on the
actual output) in order to obtain a JSON output which will contain nodes and road information, as well as the AADT calculated for each road.
Run via:
```bash
python3 QP/QP.py \
    --input $PREPROCESSING_OUTPUT_FILEPATH \
    --output $OUTPUT_JSON_FILEPATH
```


Assuming an output being *test/example1.json*, run via:

```bash
python3 QP/QP.py \
    --input test/example1.json \
    --output test/output1.json
```


Use the *--verbose* flag to show a map of each road, colorcoded depending on the AADT value.


If requesting information about how the flags in more detail, run:
```bash
python3 QP/QP.py --help
```

The solver has been found to return inaccurate results when enforcing strict constraints when setting *adaptive_rho = False*[4]. Therefore,
this has not been strictly enforced. Instead, if any negative values are found, these are solved using a second QP optimizer with the
same constraints, but which attempts to reduce the sum of squared errors with respect to the values generated by the first optimizer.


### Multi-map preprocessing and optimization

It is possible to rapidly generate as many copies as desired of a map, each with the AADT values generated randomly using the original AADT
values as the mean and standard deviation of a normal distribution. This is useful in order to rapidly generate input data for neural
networks or any other algorithm.

This can be done through the bash script *generate_multiple_QP_solutions.sh* via:

```bash
./generate_multiple_QP_solutions.sh $OSM_FILE_FILEPATH $AADT_JSON_FILEPATH $OUTPUT_JSON_FILEPATH $num_copies $multi_processed_dir
```

where the *$multi_processed_dir* represents the directory output location for the AADT values including the final '/', the final output will
be stored in a directory of the same name but ending in *_output*.


For example:
```bash
./generate_multiple_QP_solutions.sh examples/Niles/Niles.osm examples/Niles/Niles_AADT.json test/example1.json  20 test/multiple/

```

### Running the neural network

Simply install the requirements by running `pip install requirements.txt`,
then run the network with 
```bash
python3 nn/main.py --epochs 10 --model gcn --data-folder <path-to-data-folder>
```

The path to the data folder points to a folder created by the qp solver with subfolders "multiple" and "multiple_output".

For other command options run `python nn/main.py --help`.



# Licensing

This project utilizes the OSPQ solver, which uses the [Apache License 2.0](https://github.com/oxfordcontrol/osqp/blob/master/LICENSE), a copy of this license is also provided in this repository [here](./licensing/Apache_license_2.txt).


## Data examples

The provided examples were obtained using data from:
* Niles
	* AADT values obtained from [Michigan DOT 2013 ADT](https://mdotcf.state.mi.us/public/maps_adtmaparchive/listfiles.cfm?folder=2013adt)
	* Coordinate box: {"N": 41.8751, "W":-86.3211, "S":41.7903, "E":-86.1709}
	* AADT coordinates obtained from OpenStreetMap






Map data copyrighted OpenStreetMap contributors and available from [https://www.openstreetmap.org](https://www.openstreetmap.org).



# References

1. [https://stackabuse.com/reading-and-writing-xml-files-in-python/](https://stackabuse.com/reading-and-writing-xml-files-in-python/)
2. [https://stackoverflow.com/questions/17390166/python-xml-minidom-get-element-by-tag-in-child-node](https://stackoverflow.com/questions/17390166/python-xml-minidom-get-element-by-tag-in-child-node), question
3. [https://mdotcf.state.mi.us/public/maps_adtmaparchive/listfiles.cfm?folder=2013adt](https://mdotcf.state.mi.us/public/maps_adtmaparchive/listfiles.cfm?folder=2013adt)
4. [https://github.com/oxfordcontrol/OSQP.jl/issues/47](https://github.com/oxfordcontrol/OSQP.jl/issues/47)
5. [https://github.com/noderod/City-Learning/blob/master/NY_fire_inspection/NY_Visualizer.py](https://github.com/noderod/City-Learning/blob/master/NY_fire_inspection/NY_Visualizer.py)
6. [https://www.rapidtables.com/web/color/RGB_Color.html](https://www.rapidtables.com/web/color/RGB_Color.html)
7. [https://numpy.org/doc/stable/reference/generated/numpy.searchsorted.html](https://numpy.org/doc/stable/reference/generated/numpy.searchsorted.html)
8. [https://stackoverflow.com/questions/6715442/how-to-add-matplotlib-colorbar-ticks](https://stackoverflow.com/questions/6715442/how-to-add-matplotlib-colorbar-ticks), using *Eryk Sun*'s answer
9. [https://www.cyberciti.biz/faq/howto-check-if-a-directory-exists-in-a-bash-shellscript/](https://www.cyberciti.biz/faq/howto-check-if-a-directory-exists-in-a-bash-shellscript/)
10. [https://www.cyberciti.biz/faq/bash-loop-over-file/](https://www.cyberciti.biz/faq/bash-loop-over-file/)
11. [https://stackoverflow.com/questions/965053/extract-filename-and-extension-in-bash](https://stackoverflow.com/questions/965053/extract-filename-and-extension-in-bash), using *Tomi Po*'s answer
12. [https://stackoverflow.com/questions/18668556/how-to-compare-numbers-in-bash](https://stackoverflow.com/questions/18668556/how-to-compare-numbers-in-bash), using *Gabriel Staples* and *jordanm*'s answer
13. [https://www.cs.ubc.ca/~murphyk/Teaching/CS540-Fall08/hw2.pdf](https://www.cs.ubc.ca/~murphyk/Teaching/CS540-Fall08/hw2.pdf)
14. [https://www.tutorialspoint.com/numpy/numpy_inv.htm](https://www.tutorialspoint.com/numpy/numpy_inv.htm)
15. [https://matplotlib.org/stable/api/\_as_gen/matplotlib.pyplot.ylim.html](https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.ylim.html)
16. [https://wiki.python.org/moin/UsingPickle](https://wiki.python.org/moin/UsingPickle)