# UrbanScannerAnnotation: Image processing

## Architecture for semantic segmentation
Paper: [Full-Resolution Residual Networks for Semantic Segmentation in Street Scenes](https://arxiv.org/abs/1611.08323)    
Github repository: [FRRN](https://github.com/hiwonjoon/tf-frrn)      
    
This approach trained and tested on [CityScapes Dataset](https://www.cityscapes-dataset.com/dataset-overview/), which includes 30 classes:  

Group | Classes
--- | --- 
flat | `road` · `sidewalk` · parking · rail track
human | `person` · `rider`
vehicle | `car` · `truck` · `bus` · `on rails` · motorcycle · bicycle · caravan · trailer
construction | `building` · wall · fence · guard rail · bridge · tunnel
object | pole · pole group · traffic sign · traffic light
nature | `vegetation` · `terrain`
sky | `sky`
void | ground · dynamic · static

The classes in grey boxes need to be labelled in our scenario. Here are the details on definition ([Reference](https://www.cityscapes-dataset.com/dataset-overview/)):

+ `road`
    + Part of ground on which cars usually drive, 
    + i.e. all lanes, all directions, all streets. Including the markings on the road.
    + Areas only delimited by markings from the main road (no texture change) are also road, e.g. bicycle lanes, roundabout lanes, or parking spaces. This label does not include curbs.
+ `sidewalk`
    + Part of ground designated for pedestrians or cyclists.
    + Delimited from the road by some obstacle, e.g. curbs or poles (might be small), not only by markings.
    + Often elevated compared to the road. Often located at the sides of a road. This label includes a possibly delimiting curb, traffic islands (the walkable part), or pedestrian zones (where usually cars are not allowed to drive during day-time).
+ `person`
    + A human that satisfies the following criterion. Assume the human moved a distance of 1m and stopped again. If the human would walk, the label is person, otherwise not. 
    + Examples are people walking, standing or sitting on the ground, on a bench, on a chair. This class also includes toddlers, someone pushing a bicycle or standing next to it with both legs on the same side of the bicycle.
    + This class includes anything that is carried by the person, e.g. backpack, but not items touching the ground, e.g. trolleys.
+ `rider`
    + A human that would use some device to move a distance of 1m. 
    + Includes, riders/drivers of bicycle, motorbike, scooter, skateboards, horses, roller-blades, wheel-chairs, road cleaning cars, cars without roof. 
    + Note that a visible driver of a car with roof can only be seen through the window. Since holes are not labeled, the human is included in the car label.
+ `car`
    + Car, jeep, SUV, van with continuous body shape, caravan, no other trailers. 
+ `truck`
    + Truck, box truck, pickup truck. Including their trailers.
    + Back part / loading area is physically separated from driving compartment.
+ `bus`
    + Bus for 9+ persons, public transport or long distance transport.
+ `on rails`
    + Vehicle on rails, e.g. tram, train.
+ `building`
    + Building, skyscraper, house, bus stop building, garage, car port. 
    + If a building has a glass wall that you can see through, the wall is still building. Includes scaffolding attached to buildings.
+ `vegetation`
    + Tree, hedge, all kinds of vertical vegetation. 
    + Plants attached to buildings are usually not annotated separately and labeled building as well. If growing at the side of a wall or building, marked as vegetation if it covers a substantial part of the surface (more than 20%).
+ `terrain`
    + Grass, all kinds of horizontal vegetation, soil or sand. These areas are not meant to be driven on. This label includes a possibly delimiting curb. Single grass stalks do not need to be annotated and get the label of the region they are growing on.
+ `sky`
    + Open sky, without leaves of tree. Includes thin electrical wires in front of the sky.

###### Please also note the labeling policy:   
Labeled foreground objects must never have holes, i.e. if there is some background visible ‘through’ some foreground object, it is considered to be part of the foreground. This also applies to regions that are highly mixed with two or more classes: they are labeled with the foreground class. Examples: tree leaves in front of house or sky (everything tree), transparent car windows (everything car).

## Labelling tool for pixel-level semantic segmentation
Github repository: [django-labeller](https://github.com/Britefury/django-labeller)  
    
**Installation recommendation**:    
1. Clone the repository: 
```
> git clone https://github.com/Britefury/django-labeller.git
> python setup.py install
```
2. Label setting:   

Copy `schema.json` from this repository to the image folder.
   
3. Optional (only if automatic detection is needed):
   + [PyTorch](https://pytorch.org/) (CPU version is ok)
   + [DEXTR](https://github.com/Britefury/dextr) assisted labelling (recognize objects automatically) 

4. Go to the directory and run the app  

If DEXTR and PyTorch are installed:
```
 python -m image_labelling_tool.flask_labeller --enable_dextr --images_dir=<image_directory> --images_pat=*.jpeg
```
else:
```
 python -m image_labelling_tool.flask_labeller --images_dir=<image_directory> --images_pat=*.jpeg
```

5. Run:

Open `http://127.0.0.1:5000/` on the browser, go into 'Labelling tool' and follow the instructions (red button on the top).  

If DEXTR is enabled, it will use the ResNet-101 based DEXTR model trained on Pascal VOC 2012 that is provided by the dextr library.   
Thus, the tool can roughly identify the following objects: 

Group | Classes
--- | ---
Person | `person`
Animal | bird, cat, cow, dog, horse, sheep
Vehicle | aeroplane, bicycle, boat, `bus`, `car`, motorbike, `train`
Indoor | bottle, chair, dining table, potted plant, sofa, tv/monitor


## Example
![Example](https://github.com/MingZx8/UrbanScannerAnnotation/blob/main/example/Example.png)



## Sampling
+ 1.5 images/hour (fine annotation)
+ 3+ images/hour (coarse annotation)
  
+ Number of images? 
  + 400-600 (20% for validation, 80% for training)   

+ Location (Route)
  
+ Route type
  + Main road
  + Minor road
  + Highway
  + ...
  
+ Weather
  + Sunny
  + Rainy
  + Cloudy
  + ...

+ Time
  + Morning
  + Noon
  + Afternoon

## Annotation format
The Django-Labeller generates .json file that looks like:    
```
{
  "image_filename": "camera1599829394413.jpeg",
  "completed_tasks": [
    "finished"
  ],
  "labels": [
    {
      "label_type": "polygon",
      "label_class": "car",
      "source": "manual",
      "anno_data": {},
      "regions": [
        [
          {
            "x": 35.68219718867216,
            "y": 100
          },
          {
            "x": 35.0579373995799,
            "y": 100
          },
          ...
        ]
      ],
      "object_id": "58aeb42d-47be-4d5e-a5ae-c2f492719b6c__1"
    },
    ...
```
The CityScape Datasets provides .json file that looks like:     
```
{
    "imgHeight": 1024, 
    "imgWidth": 2048, 
    "objects": [
        {
            "label": "sky", 
            "polygon": [
                [
                    64, 
                    0
                ], 
                [
                    99, 
                    74
                ], 
                ...
            ]
        }, 
        ...
```
It should determine what format to use in our case.
