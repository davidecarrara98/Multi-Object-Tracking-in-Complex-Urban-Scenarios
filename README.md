# Multi Object Tracking in Complex Urban Scenarios - Project Course in Mathematical Modelling

<p align="center">
    <img src="https://user-images.githubusercontent.com/65012600/156659204-10fb924f-f431-4f92-bcf1-ba12afb381b2.png" width="180" alt="Chalmers"/>
    <img src="https://user-images.githubusercontent.com/65012600/156658116-93891be7-299e-4d87-89bb-c465d62588c2.png" width="220" alt="Viscando"/>
</p>

## Overview

Object tracking is the process of detecting and following moving entities in subsequent observations from multiple sensors.
Accurate tracking in complex urban scenarios is crucial for safety applications such as accident mitigation, predictive traffic control and design of safer infrastructure.
Despite a well-established theoretical framework for object tracking, various challenges need to be addressed in a real context, including sensor noise, occlusions and asynchronous measures from multiple sensor.

<p align="center">
    <img src="https://user-images.githubusercontent.com/65012600/156663487-9634b5a5-c8d9-46d0-bf83-b270ec062e1d.png" width="400" alt="Scenario"/>
</p>


Viscando uses proprietary stereovision sensors to extract detailed traffic movement information in critical urban settings, and algorithms to produce insights on mobility risks, communicate real-time safety information and study traffic scenarios and behavior models. One key feature of these vision tools is the ability to correctly identify and locate objects, and track their movement in consecutive measurements or partially overlapping views. The aim of the present work is to provide an algorithm to unify detections into tracks, with a focus on accuracy and smoothness of the output, as well as providing a flexible implementation that may easily be modified and re-used.

## Dataset

* Provided data is a collection of information from 4 stereovision sensors, with detections corresponding to approximately 750 frames or one minute of activity, of a roundabout in Kolliken, Switzerland. Data has been previously processed to provide points in 3D as centers of detected objects, and bounding boxes containing those objects. In the particular case, the data was collected in Switzerland where video collection for development purposes was not prohibited at the time of collection. Collected video (consisting of image frames) is solely used for research and improvement of object detection and tracking algorithms.

<p align="center">
    <img src="https://user-images.githubusercontent.com/65012600/156665237-90994fc6-a4b6-403c-806f-52eaa0b17128.png" width="300" alt="Chalmers"/>
    <img src="https://user-images.githubusercontent.com/65012600/156665317-3f22d3f8-8f50-4341-b1e9-4660e2004038.png" width="300" alt="Viscando"/>
</p>


## Main Challenges and Solutions

* It can happen that an object detected in a certain frame is occluded in the following ones, resulting in uncertainties about the position of the object. In addition to this, the sensors may fail in associating only one detection per frame to an object. The problem of multiple detections is very common in the data set, and can cause several tracks to be associated to a single object. It is thus important to understand which detections to retain.


<p align="center">
    <img src="https://user-images.githubusercontent.com/65012600/156666012-e4883c3d-ea7a-4b48-bf65-7f18f87a6c5f.png" width="300" alt="Multiple"/>
    <img src="https://user-images.githubusercontent.com/65012600/156666142-a7fc0e4f-c246-4cdc-b5c2-350e040bce67.png" width="300" alt="Occlusion"/>
</p>

* A first solution to the problem of occlusion and multiple detections was found by eliminating detections from areas where sensors could not be considered reliable. This areas differ for each sensor, and usually depend on the presence of obstacle or on the distance from the sensor itself. The detections in zones indicated in the following pictures are removed from the dataset, considering the fact that they are always better captured from a better positioned sensor.


<p align="center">
    <img src="https://user-images.githubusercontent.com/65012600/156666738-8361ed8a-0cd9-42d3-b98d-7f081c5eb788.png" width="619" alt="princing no seasonal"/>
</p>
* A following minor tweak was found by appliying an horizontal and then radial transformation on the detections provided by the third sensor. The transformation was discovered by another group working on the same dataset but with different goals, and slightly improved our results.

<p align="center">
    <img src="https://user-images.githubusercontent.com/65012600/156667675-5c3ad6dd-b636-418c-b770-73705ce67914.png" width="417" alt="princing no seasonal"/>
</p>

* Lastly, we implemented a rough separation of the surfaces belonging to pavement and road, in order to distinguish between detections likely related to pedestrians and thw ones originated by vehicles. This approach can also be useful for a posteriori analysis of curves


<p align="center">
    <img src="https://user-images.githubusercontent.com/65012600/156668130-4ecdb5ac-5c67-4212-adf0-87f0ae67f6c1.png" width="417" alt="princing no seasonal"/>
</p>

## The Tracker

The tracker represents the main part of the algorithm, which receives as input all the accepted detections and returns the tracks connecting the same object across the frames.
The main protagonist of the tracker is the Kalman Filter, which is used to predicted the next steps of a track from the previous position and is able to filter the noise due to errors in the detections. Its tuning and building consists of an important part of the project.

* At first detections from different cameras are joined according to an internal clock, considering as synchronous detections collected in an interval of 0.08 seconds. IN order to join the detections related to the same object from different sensors, we center a gaussian distribution in each object, using the bounding box to determine covariance axis, and compute the probability of representing the same object for each couple of detections.


<p align="center">
    <img src="https://user-images.githubusercontent.com/65012600/156668934-8c202d9f-d24d-450a-8581-94ef2192dc53.png" width="488" alt="princing no seasonal"/>
</p>

* For each detection, we compute the distance with the last instance of every track. This distance is given by the weighted sum of euclidean distance and of 1 - Intersection over Union of the bounding boxes. The latter is particularly useful during the first stage of movement, where the Kalman Filter has not yet converged and the bounding boxes provide a good intuition of the trajectory.


<p align="center">
    <img src="https://user-images.githubusercontent.com/65012600/156669753-73b6853b-7de1-481e-9a4a-199878f54fbf.png" width="617" alt="bound box"/>
</p>

* The association between the detections and tracks is then performed through a visiting algorithm, typical of matching problems. The specifics of the algorithm can be found in greater details in the final report, and theoretically assure the best possible matching for the tracks.

* Tracks are then classified according to their status. For each matched track the filtered or predicted position is then saved as the track true position.

*  Active and Pending (represented object that are likely to be temporarily occluded) tracks are propagated using the filter, with a simple but effective motion model with constant acceleration.

## Results

The final result of the algorith consisted in 154 identified tracks, with 198 detections linked in the longest one. 210 tracks were instead removed, since deemed irrelevant by the tracker. In the following picture there is an overview of the obtained tracks, where the shape of the roundabout is clearly visible.

<p align="center">
    <img width="398" alt="overview" src="https://user-images.githubusercontent.com/65012600/157850867-0ed2add4-a65c-4566-8cbd-78d59654d050.png", alt = 'overview'>
</p>

Two examples of tracks are reported in the figure below, one corresponding to a vehicle moving around all the roundabout, the second to a pedestrian crossing the road. Both tracks are a good example of the tracking properties of the algorithm. More insights about these tracks, and analyses of the velocities are provided in the report.
<p align="center">
    <img width="200" alt="long track" src="https://user-images.githubusercontent.com/65012600/157852358-016e88af-ca1a-40bb-9f75-0fc524eaf526.png", alt = "long">
    <img width="200" alt="pedestrian" src="https://user-images.githubusercontent.com/65012600/157853391-efe11047-0f15-46ce-a19d-77c44828a49b.png", alt = "pedestrian">

</p>

## Resources
- ```Main.ipynb``` is the Jupyter Notebook containing all the work of the project. The results are displayed, but the code cannot be used since the data are private.
- ```Final Report``` is the pdf of the final report of the project. It contains detailed information on every step of the algorithm, a more accurate analysis of challenges and results and some theoretical insights.

## Team

- Davide Carrara [[Github](https://github.com/davidecarrara98)] [[Email](mailto:davide1.carrara@mail.polimi.it)]
- Lupo Marsigli [[Github](https://github.com/LupoMarsigli)] [[Email](mailto:lupo.marsigli@mail.polimi.it)]
- Francesco Romeo [[Github](https://github.com/fraromeo)] [[Email](mailto:francesco5.romeo@mail.polimi.it)]
- Valentina Sgarbossa [[Github](https://github.com/vale9888)] [[Email](mailto:valentina.sgarbossa@mail.polimi.it)]
