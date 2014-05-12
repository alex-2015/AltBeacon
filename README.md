AltBeacon
=========

AltBeacon is an alternative to iBeacon that allow the iPhone to be advertised in the background, which is not currently possible in iBeacon. It is based on the open source project Vinicity (thanks Ben Ford) https://github.com/Instrument/Vicinity. By advertising in the background a whole new range of use cases are possible, like for exmaple a messaging app for nearby people. We are currenlty using this framework to develop a product that will be soon in the AppStore. 


How does it work
----------------

The key behind AltBeacon is that the Bluetooth Low Energy stack of iOS allows background advertising when acting as a Peripheral (In android this is still not possible but we are working in a workaround). However, when advertising in background it is only possible to discover UUIDs that you know previously (this is mentioned in the BLE documentation of apple but it was also tested by us). So the following call:

    NSDictionary *scanOptions = @{CBCentralManagerScanOptionAllowDuplicatesKey:@(YES)};
    [centralManager scanForPeripheralsWithServices:nil options:scanOptions];

Will find all the beacons around that are advertising in foreground but none that is advertising in background. 
This is a problem to make scalable as if you would have a database with 1000000 AltBeacons it would be impossible to scan them all. The trick here sis to use Location to define an radius to look for and then search for the AltBeacons in that area. Our test show that you can scan for maximum 25 UUIDS at a time. If you scan for more than that the CentralManager would return crap. The following call, if it contains less than 25 UUIDS, will find them correctly.

    NSDictionary *scanOptions = @{CBCentralManagerScanOptionAllowDuplicatesKey:@(YES)};
    [centralManager scanForPeripheralsWithServices:uuidsToDetect options:scanOptions];

Consering that it takes just a few seconds in the worst case to find the beacons in less than a minute you can scan for hundred of beacons, and if in addition you limit the number of AltBeacons using Location to a radius of a couple of kilometers then this system is quite scalable. 

Version
----

0.1

Installation
----

Copy the source folder into your xcode project. (Soon I will add a cocoapod as well)

Usage
----

There is a Demo project inside the source code. Please check it.

Define the Uuids of the AltBeacons. In a real project you would generate this automatically and store them in a database to then perform the locations radius filtering. 

    #define kUuidBeaconOne @"5F22CA05-8F6C-49B6-AEAE-B278FDFE9287"
    #define kUuidBeaconTwo @"9F3E9E58-5073-4F78-BD04-87050DAFB604"
    #define kUuidBeaconThree @"177383C7-8347-444F-B14E-1581131A16E2"


Then start create the beacons broadcasting and detecting. 

    // Initialize the IBeacon UUDI@
    self.beaconOne =  [[AltBeacon alloc ]initWithIdentifier:kUuidBeaconOne clearFoundDevicesInterval:CLEAR_INTERVAL];
    self.beaconTwo =  [[AltBeacon alloc ]initWithIdentifier:kUuidBeaconTwo clearFoundDevicesInterval:CLEAR_INTERVAL];
    self.beaconThree =  [[AltBeacon alloc ]initWithIdentifier:kUuidBeaconThree clearFoundDevicesInterval:CLEAR_INTERVAL];
    [self.beaconOne addDelegate:self];
    [self.beaconTwo addDelegate:self];
    [self.beaconThree addDelegate:self];
    
    // Add the beacons to the array
    self.uuidsToSearch = [[NSMutableArray alloc]initWithObjects:[CBUUID UUIDWithString:kUuidBeaconOne],[CBUUID UUIDWithString:kUuidBeaconTwo],[CBUUID UUIDWithString:kUuidBeaconThree], nil];

Then tell the beacon to start detecting and advertising (broadcasting). You need to pass the uuids to detect. As we mention, pass 25 maximum per detection cycle and use Location and a uuids database to filter to the uuids of the devices in a radius of a few km to make this approach scalable. 

    - (void)start:(AltBeacon *)beacon {

        // start broadcasting
        [beacon startBroadcasting];
        [beacon startDetecting:self.uuidsToSearch];
    }

    - (void)stop:(AltBeacon *)beacon {
        
        // start broadcasting
        [beacon stopBroadcasting];
        [beacon stopDetecting];
    }

    
Implement the delegate to receive the information when the devices are found. You receive a dictionary with the uuid as a key and the range and enum with Immediate, Near and Far as the value. The range indicates how close you are to the uuids that was detected. This callback is called once a second and the list of uuids find is cleared every a few seconds. You can define the clearance time in the AltBeacon constructor. 

    // Delegate methods
    - (void)service:(AltBeacon *)service foundDevices:(NSMutableDictionary *)devices {
        
        if (devices.allKeys.count > 0){
            for(NSString *key in devices) {
                NSNumber * range = [devices objectForKey:key];
                NSString *result = [self convertToString:range];
                NSString *beaconName = @"";
                if ([key  isEqualToString:kUuidBeaconOne]){
                    beaconName = @"Beacon one!";
                    self.labelDisplayResultBeacon1.text = [NSString stringWithFormat:@"%@ %@ %@ %@", beaconName, @"was found",result, @"meters away"];
                }
                else if ([key  isEqualToString: kUuidBeaconTwo]){
                    beaconName = @"Beacon two!";
                    self.labelDisplayResultBeacon2.text = [NSString stringWithFormat:@"%@ %@ %@ %@", beaconName, @"was found",result, @"meters away"];
                }
                else if ([key  isEqualToString: kUuidBeaconThree]){
                    beaconName = @"Beacon three!";
                    self.labelDisplayResultBeacon3.text = [NSString stringWithFormat:@"%@ %@ %@ %@", beaconName, @"was found",result, @"meters away"];
                }
            }
        }else{
            self.labelDisplayResult.text = @"Scanning...............";
            self.labelDisplayResultBeacon1.text = @"";
            self.labelDisplayResultBeacon2.text = @"";
            self.labelDisplayResultBeacon3.text = @"";
        }
    }

Other things
----
Please notice that this library is under development, so please feel free to contribute and mention design alternatives and possibilities and contacts us at charrualabs@gmail.com

License
----

MIT

    