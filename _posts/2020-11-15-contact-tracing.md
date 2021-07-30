---
layout: inner
title: 'Contact Tracing with iBeacons'
date: 2020-11-15 12:00:00
categories: iBeacons Firebase iOS ML
type: project
tags: SWIFT Firebase iBeacon
featured_image: '/project/img/posts/contact-tracing/contact-tracing.png'
project_link: 'https://github.com/haaaamza/ECSE-682/tree/master/final%20project'
button_icon: 'github'
button_text: 'Visit Project'
lead_text: 'Using Cloud and iBeacons to detect potential exposure to COVID.'
author: Hamza

---
### Preface
This app was built with Jack Hu at the McGill University Integrated Microsystems Lab. The Integrated Microsystems Lab at McGill (IML) ([link](http://www.iml.ece.mcgill.ca/index_iml.php)) deals with the applications of embedded systems to real life.
# The Proposed Solution

The goal of this project was to develop application that acts as a universal Contact tracer, by obtaining data from another iBeacon devices using Bluetooth Low Energy and storing it in the cloud, the system is able to determine the location of the user at a specific time.
The application allows for a positive test result to be added through the app, this updates the database for all users within the same location & time and notifies them of a potential exposure.

Given the scenario where people remain confined in small spaces (offices, classrooms) for long periods, the CDC has stated that maintaining a distance of 2 metres is insufficient in preventing a Covid-19 infection [^1]. Therefore, the app uses a radius of 6-metres, and a 5 minute window.

## Contact Tracing System
The Contact tracing System follows the following steps:
1. App scans for iBeacons
2. Detected information is added to Cloud (device information, timing)
3. User adds positive results, this is confirmed by text recognition
4. Cloud is updated to reflect positive result status
5. Other Users in Vicinity within a Time-range are notified of Positive Result 
6. UI is updated for all affected users

The infrastructure is comprised of 3 major processing endpoints:
  1. iBeacon 
  2. Mobile App
  3. Firebase
  
![img1](/project/img/posts/contact-tracing/flowchart.png){:height="576" width="239"}
### The iBeacon
The board used for iBeacon simulation is the Thunderboard RS9116 WiSeConnect. The RS9116 module uses Bluetooth Low Energy to advertise as an iBeacon and UART communication via teraterm to configure the module as an iBeacon.

The contact tracing app scans for the RS9116 beacons around which advertise their UUIDs, major, minor values. This helps the iOS device to calculate the location of a user from the beacon device. Figure 2 shows a sample iBeacon infrastructure that can be used indoors in order to find the location of the user.

![img2](/project/img/posts/contact-tracing/table.png){:height="576" width="186"}

As shown in the figure above the iBeacon UUID, Major & Minor values can help pinpoint the exact location of the user. In this case the user is a student, this infrastructure is useful for contact tracing in small and enclosed spaces.

### The App

The app is made using SWIFT and Core Location. 
The app has 3 view controllers:
	1. Main VC
		• It displays the status of potential infection to the user. If the user has been potentially infected, or tested positive this main view controller changes.
	2. Self-Report VC
			• Upload COVID test result -> verified by text detection.
			• Calls function to update Firebase of positive result.
	3. Information VC
			• Sends user to WHO Website that gives information on COVID-19

| ![img3](/project/img/posts/contact-tracing/ui1.png){:height="591" width="332"} | ![img4](/project/img/posts/contact-tracing/ui2.png){:height="591" width="332"} |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![img5](/project/img/posts/contact-tracing/ui3.png){:height="591" width="332"} | ![img6](/project/img/posts/contact-tracing/ui4.png){:height="591" width="332"} |

The iBeacon tells the application the percieved distance of the user from the iBeacon. This is classified in three states: immediate, near, far. Being near/immediate means the user is within 3 metres of that particular iBeacon. 
If the user is near or immediate to a particular iBeacon, the app will write to Firebase a struct containing:
1. Randomly generated phone id that allows for privacy between the user and the database.
2. major, minor and UUID value of the iBeacon. This helps us determine the location of the user.
3. A test result boolean that keeps track of user exposure to the disease
4. A proximity enumeration that is expressed as an integer helps us keep track of the approximate distance between the user and the iBeacon, and a distance that a more accurate distance measurement in metres between the user and the iBeacon
5. A @DocumentID variable that keeps track of the specific interaction between the user and the iBeacon and a @ServerTimestamp variable that tracks the time the document was added to the database

The information in the struct will ensure that if a user tests positive, all potentially exposed users can be notified. The struct will be able to pinpoint which users are within a 3 metre radius of the infected user within the same time frame. This is depicted in the figure below.

![img7](/project/img/posts/contact-tracing/distance.png)

### Firebase
We use a SnapshotListener that takes a Snapshot of the database, and runs selected code if the database has changed. A SnapshotListener is basically an observer, that notifies the application that data has changed. 

In our case the SnapshotListener picks up on a database change, and checks if the COVID test result is true. The control flow is shown below:
1. User 1 Uploads positive result
2. User 1’s application updates Firestore Database for other Users
  • All users in vicinity have their documents updated in the Firestore
  • Users within 6 Metres, and a 5 minute window are now 'exposed'
3. The SnapshotListener for each affected user is called since the Firestore has been changed.
	• Update the App UI notifying them of the COVID exposure.
Firestore is scalable, it enables the application to support a growing population of users.[^2]

The figure below shows the Firestore setup. Each seperate iBeacon will have a seperate subcollection that records all interactions between users. 
![img8](/project/img/posts/contact-tracing/firestore.png)

## Privacy
Institutions such as provincial governments, schools, and companies can create unique accounts for their users. These accounts serve as a medium for authentication as Firebase Cloud has security rules that grant developer/authenticated accounts to access the remote storage.
A proposed solution that can protect the identity of users from third parties and also keep a layer of privacy from institutions:
	1) Authentication from central entity to write to the Cloud 
	2) Anonymous device IDs are generated locally from the device application and are used for Cloud functions and beacon contact tracing.
The proposed solution above prevents malicious entities from accessing and altering sensitive information regarding the users’ health status. The device identifiers anonymously help contact trace which devices were present near particular iBeacons at a particular time. This two-step solution allows for not only secure cloud access but also maintains users’ anonymity within institutions like schools, offices, governments, etc...


## Conclusion
Contact tracing is a necessary step in managing the spread of pandemic diseases in the foreseeable future. This project evaluates the use of embedded systems in contact tracing. The solution offers an effective method to build contact tracing infrastructure within settings such as schools and office buildings.
At the end the team was satisfied by the functionality of the app. There were some improvements that would have increased the utility of the app, such as a Notification of exposure sent to the user directly and creating possible integration solutions to work with Government healthcare systems.



------

**You can see the final paper [here](https://drive.google.com/file/d/18gY8k0e6w29lctstFQHlq7tn0mEuCGwQ/view){:target="_blank"}**


[^1]:”Scientific Brief: SARS-CoV-2 Transmission”, Centers for Disease Control and Prevention, 2021. [Online]. Available: https://www.cdc.gov/coronavirus/2019-ncov/science/science-briefs/sars- cov-2-transmission.html. [Accessed: 19- December- 2021].

[^2]:Cloud Firestore — Firebase, Google. [Online]. Available: https://firebase.google.com/docs/firestore. [Accessed: 02-Dec-2020].

```
