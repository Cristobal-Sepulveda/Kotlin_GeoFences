________________________________________________________________________________

                            1. Introduction
______________________________________________________________________________

A geofence is a virtual perimeter defined by GPS or RFID around a real
world area. Geofences can be created with a radius around a point location.
Geofencing has a lot of applications including:

    * Reminder apps, such as reminding you to pick up a prescription when
      you get to a certain destination like your pharmacy.

    * Child location services, where a parent can be notified if a child
      leaves an area designated by a geofence.

    * Attendance, where an employer can know when their employees arrive
      by the time they enter a geofence.

    * A treasure hunt app that uses geofences to mark the place where the
      treasure is hidden, when you enter that place you will be notified
      that you have won! - this is what you will be making in this lesson!

Geofences have three transition types:

https://video.udacity-data.com/topher/2019/November/5dcc7f79_image4/image4.png

  Enter: Indicates that the user entered the geofence(s).

  Dwell : Indicates that the user enters and dwells in geofences for a given period of time.

  Exit: Indicates that the user has exited the geofence(s).

https://video.udacity-data.com/topher/2019/November/5dcc8008_image5/image5.png
This shows geofence locations denoted by markers and the radiuses around them.

What you'll need

    * The latest version of Android Studio
    * A minimum of SDK API 29 on your device or emulator. (This should still
      work on lower API levels but may look different)

What you'll learn

    * How to check user permissions.

    * How to check device settings.

    * How to add Broadcast Receivers.

    * How to add geofences.

    * How to handle geofence transitions.

    * How to mock locations in the emulator.

Reference Documentation

Geofencing
https://developers.google.com/location-context/geofencing
Create and monitor geofences
https://developer.android.com/training/location/geofencing
________________________________________________________________________________

                            2. Run the Starter code
________________________________________________________________________________

  You can either download all the sample code to your computer...

  Download Zip

  ...or clone the GitHub repository from the command line and
  checkout the starter branch.

    $ git clone https://github.com/udacity/android-kotlin-geo-fences.git

  The starter app contains code to help you get started. It contains some
  assets, layouts, the opening activity and a file with services that
  you will complete during this lesson.

Important classes provided for you:

    * HuntMainActivity.kt is the main class you will be working in. This class
      contains skeleton code for functions that handle permissions, adding
      geofences and removing geofences.

    * GeofenceViewModel.kt is the ViewModel associated with HuntMainActivity.kt.
      This class handles the LiveData and determines which hint should be
      shown on screen.

    * NotificationUtils.kt: When you enter a geofence, a notification
      pops up! This class creates and styles that notification.

    * activity_main.xml: This currently displays an image of an Android but
      you will implement it to display a hint.

    * GeofenceBroadcastReceiver.kt: This contains skeleton code for
      the onReceive() method of the BroadcastReceiver where you will handle

1. Run the app on the emulator or on your own device, you should see the
   splash page.

https://video.udacity-data.com/topher/2019/November/5dcc81b6_image6/image6.png
________________________________________________________________________________

                            3. Request Permissions
________________________________________________________________________________

Add permissions to the AndroidManifest

  The Geofencing API requires location to be shared at all times. If you are
  on Q or later, you will need to specifically ask the user for it.
  You will now add in the location permissions in the AndroidManifest.xml:

    1. Add in permissions for ACCESS_FINE_LOCATION and ACCESS_BACKGROUND_LOCATION
       permission above the application tag.

          <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
          <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />

Check if the device is running Q (API 29) or later

  The difference in location permissions only affects Android Q or
  later, so the background location permission is only needed for those devices.

    1. In HuntMainActivity.kt, add a member variable above the onCreate() method
       called runningQOrLater. This will check what API the device is running.

          private val runningQOrLater = android.os.Build.VERSION.SDK_INT >=
          android.os.Build.VERSION_CODES.Q

Create method to check for permissions

  In your app, you will need to check if permissions are granted, and if
  they are not, ask for the correct permissions.

    1. In HuntMainActivity, replace the code in the foregroundAndBackgroundLocationPermissionApproved method with this.
    Each step will be explained in the bullet points below:

          @TargetApi(29)
          private fun foregroundAndBackgroundLocationPermissionApproved(): Boolean {
             val foregroundLocationApproved = (
                     PackageManager.PERMISSION_GRANTED ==
                     ActivityCompat.checkSelfPermission(this,
                         Manifest.permission.ACCESS_FINE_LOCATION))
             val backgroundPermissionApproved =
                 if (runningQOrLater) {
                     PackageManager.PERMISSION_GRANTED ==
                     ActivityCompat.checkSelfPermission(
                         this, Manifest.permission.ACCESS_BACKGROUND_LOCATION
                     )
                 } else {
                     true
                 }
             return foregroundLocationApproved && backgroundPermissionApproved
          }

    * First, check if the ACCESS_FINE_LOCATION permission is granted.

          val foregroundLocationApproved = (
                     PackageManager.PERMISSION_GRANTED ==
                     ActivityCompat.checkSelfPermission(this,
                         Manifest.permission.ACCESS_FINE_LOCATION))

    * If the device is running Q or higher, check that the
      ACCESS_BACKGROUND_LOCATION permission is granted. Return true
      if the device is running lower than Q where you don't need a
      permission to access location in the background.

          val backgroundPermissionApproved =
             if (runningQOrLater) {
                 PackageManager.PERMISSION_GRANTED ==
                 ActivityCompat.checkSelfPermission(
                     this, Manifest.permission.ACCESS_BACKGROUND_LOCATION
                 )
             } else {
                 true
             }

    * Return true if the permissions are granted and false if not.

          return foregroundLocationApproved && backgroundPermissionApproved

Request permissions

    1. Copy this code into the requestForegroundAndBackgroundLocationPermissions()
       method, This is where you will ask the user to grant location permissions.
       Each step will be explained in the bullet points below:

          @TargetApi(29 )
          private fun requestForegroundAndBackgroundLocationPermissions() {
             if (foregroundAndBackgroundLocationPermissionApproved())
                 return
             var permissionsArray = arrayOf(Manifest.permission.ACCESS_FINE_LOCATION)
             val resultCode = when {
                 runningQOrLater -> {
                     permissionsArray += Manifest.permission.ACCESS_BACKGROUND_LOCATION
                     REQUEST_FOREGROUND_AND_BACKGROUND_PERMISSION_RESULT_CODE
                 }
                 else -> REQUEST_FOREGROUND_ONLY_PERMISSIONS_REQUEST_CODE
             }
             Log.d(TAG, "Request foreground only location permission")
             ActivityCompat.requestPermissions(
                 this@HuntMainActivity,
                 permissionsArray,
                 resultCode
             )
          }

    * If the permissions have already been approved, you don’t need to ask again.
      Return out of the method.

            if (foregroundAndBackgroundLocationPermissionApproved())
               return

    * The permissionsArray contains the permissions that are going to be
      requested. Initially, add ACCESS_FINE_LOCATION since that will
      be needed on all API levels.

            var permissionsArray = arrayOf(Manifest.permission.ACCESS_FINE_LOCATION)

    * Below it, you will need a resultCode. The code will be different depending
      on if the device is running Q or later and will inform us if you need to
      check for one permission (fine location) or multiple permissions
      (fine and background location) when the user returns from the permission
      request screen. Add a when statement to check the version running and
      assign result code to
      REQUEST_FOREGROUND_AND_BACKGROUND_PERMISSION_RESULT_CODE if the device
      is running Q or later and
      REQUEST_FOREGROUND_ONLY_PERMISSIONS_REQUEST_CODE if not.

            val resultCode = when {
               runningQOrLater -> {
                   permissionsArray += Manifest.permission.ACCESS_BACKGROUND_LOCATION
                   REQUEST_FOREGROUND_AND_BACKGROUND_PERMISSION_RESULT_CODE
               }
               else -> REQUEST_FOREGROUND_ONLY_PERMISSIONS_REQUEST_CODE
            }

    * Request permissions passing in the current activity, the permissions
      array and the result code.

            ActivityCompat.requestPermissions(
               this@HuntMainActivity,
               permissionsArray,
               resultCode
            )

Handle permissions

Once the user responds to the permissions, you will need to handle their
response using the onRequestPermissionsResult().

    1. Copy this code in under the:

            override fun onRequestPermissionsResult(
               requestCode: Int,
               permissions: Array<String>,
               grantResults: IntArray
            ) {
               Log.d(TAG, "onRequestPermissionResult")

               if (
                   grantResults.isEmpty() ||
                   grantResults[LOCATION_PERMISSION_INDEX] == PackageManager.PERMISSION_DENIED ||
                   (requestCode == REQUEST_FOREGROUND_AND_BACKGROUND_PERMISSION_RESULT_CODE &&
                           grantResults[BACKGROUND_LOCATION_PERMISSION_INDEX] ==
                           PackageManager.PERMISSION_DENIED))
               {
                   Snackbar.make(
                       binding.activityMapsMain,
                       R.string.permission_denied_explanation,
                       Snackbar.LENGTH_INDEFINITE
                   )
                       .setAction(R.string.settings) {
                           startActivity(Intent().apply {
                               action = Settings.ACTION_APPLICATION_DETAILS_SETTINGS
                               data = Uri.fromParts("package", BuildConfig.APPLICATION_ID, null)
                               flags = Intent.FLAG_ACTIVITY_NEW_TASK
                           })
                       }.show()
               } else {
                   checkDeviceLocationSettingsAndStartGeofence()
               }
            }

  Permissions can be denied in a few ways:

    * If the grantResults array is empty, then the interaction was interrupted
      and the permission request was cancelled.

    * If the grantResults array’s value at the LOCATION_PERMISSION_INDEX has
      a PERMISSION_DENIED it means that the user denied foreground permissions.

    * If the request code equals
      REQUEST_FOREGROUND_AND_BACKGROUND_PERMISSION_RESULT_CODE and the BACKGROUND_LOCATION_PERMISSION_INDEX is denied it means that the device
      is running API 29 or above and that background permissions were denied.

            if (grantResults.isEmpty() ||
               grantResults[LOCATION_PERMISSION_INDEX] == PackageManager.PERMISSION_DENIED ||
               (requestCode == REQUEST_FOREGROUND_AND_BACKGROUND_PERMISSION_RESULT_CODE &&
                       grantResults[BACKGROUND_LOCATION_PERMISSION_INDEX] ==
                       PackageManager.PERMISSION_DENIED))

    * This app has very little use when permissions are not granted so
      present a snackbar explaining that the user needs location
      permissions in order to play.

            Snackbar.make(
               binding.activityMapsMain,
               R.string.permission_denied_explanation,
               Snackbar.LENGTH_INDEFINITE
            )
               .setAction(R.string.settings) {
                   startActivity(Intent().apply {
                       action = Settings.ACTION_APPLICATION_DETAILS_SETTINGS
                       data = Uri.fromParts("package", BuildConfig.APPLICATION_ID, null)
                       flags = Intent.FLAG_ACTIVITY_NEW_TASK
                   })
               }.show()
    * If not, permissions have been granted! Call
      the checkDeviceLocationSettingsAndStartGeofence() method!

            else {
               checkDeviceLocationSettingsAndStartGeofence()
            }

    2. Run the app! You will see a pop up prompting you to grant permissions.
       Choose Allow all the time or Allow if you are running an API lower
       than 29.

https://video.udacity-data.com/topher/2019/November/5dcc883b_image11/image11.png
________________________________________________________________________________

                            4. Check Device Location
________________________________________________________________________________

  Having the user grant permissions is only one part of the permissions
  needed, another thing to check is if the device’s location is on.
  In this step, you will add code to check that a user has their device
  location enabled and if not, display an activity where they can
  turn it on.

    1. Copy this code into the checkDeviceLocationSettingsAndStartGeofence()
       method in HuntMainActivity.kt, we will go over the steps in the bullet
       points below.

            private fun checkDeviceLocationSettingsAndStartGeofence(resolve:Boolean = true) {
               val locationRequest = LocationRequest.create().apply {
                   priority = LocationRequest.PRIORITY_LOW_POWER
               }
               val builder = LocationSettingsRequest.Builder().addLocationRequest(locationRequest)
               val settingsClient = LocationServices.getSettingsClient(this)
               val locationSettingsResponseTask =
                   settingsClient.checkLocationSettings(builder.build())
               locationSettingsResponseTask.addOnFailureListener { exception ->
                   if (exception is ResolvableApiException && resolve){
                       try {
                           exception.startResolutionForResult(this@HuntMainActivity,
                               REQUEST_TURN_DEVICE_LOCATION_ON)
                       } catch (sendEx: IntentSender.SendIntentException) {
                           Log.d(TAG, "Error getting location settings resolution: " + sendEx.message)
                       }
                   } else {
                       Snackbar.make(
                           binding.activityMapsMain,
                           R.string.location_required_error, Snackbar.LENGTH_INDEFINITE
                       ).setAction(android.R.string.ok) {
                           checkDeviceLocationSettingsAndStartGeofence()
                       }.show()
                   }
               }
               locationSettingsResponseTask.addOnCompleteListener {
                   if ( it.isSuccessful ) {
                       addGeofenceForClue()
                   }
               }
            }

    * First, create a LocationRequest, a LocationSettingsRequest Builder.

               val locationRequest = LocationRequest.create().apply {
                   priority = LocationRequest.PRIORITY_LOW_POWER
               }
               val builder = LocationSettingsRequest.Builder().addLocationRequest(locationRequest)

    * Next, use LocationServices to get the Settings Client and create
      a val called locationSettingsResponseTask to check the location settings.

              val settingsClient = LocationServices.getSettingsClient(this)
              val locationSettingsResponseTask =
                 settingsClient.checkLocationSettings(builder.build())

    * Since the case we are most interested in here is finding out if
      the location settings are not satisfied, add an onFailureListener()
      to the locationSettingsResponseTask.

              locationSettingsResponseTask.addOnFailureListener { exception ->
              }

    * Check if the exception is of type ResolvableApiException and if so, try
      calling the startResolutionForResult() method in order to prompt the user
      to turn on device location.

              if (exception is ResolvableApiException && resolve){
                 try {
                     exception.startResolutionForResult(this@HuntMainActivity,
                         REQUEST_TURN_DEVICE_LOCATION_ON)
                 }

    * If calling startResolutionForResult enters the catch block, print a log.

              catch (sendEx: IntentSender.SendIntentException) {
                 Log.d(TAG, "Error getting location settings resolution: " + sendEx.message)
              }

    * If the exception is not of type ResolvableApiException, present a
      snackbar that alerts the user that location needs to be enabled
      to play the treasure hunt.

              else {
                 Snackbar.make(
                     binding.activityMapsMain,
                     R.string.location_required_error, Snackbar.LENGTH_INDEFINITE
                 ).setAction(android.R.string.ok) {
                     checkDeviceLocationSettingsAndStartGeofence()
                 }.show()
              }

If the locationSettingsResponseTask does complete, check that it is
successful, if so you will want to add the geofence.

              locationSettingsResponseTask.addOnCompleteListener {
                 if ( it.isSuccessful ) {
                     addGeofenceForClue()
                 }
              }
    2. Replace the code below in the onActivityResult() method. After the
       user chooses whether to accept or deny device location permissions,
       this checks if the user has chosen to accept the permissions.
       If not, it will ask again.

              override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
                 super.onActivityResult(requestCode, resultCode, data)
                 if (requestCode == REQUEST_TURN_DEVICE_LOCATION_ON) {
                     checkDeviceLocationSettingsAndStartGeofence(false)
                 }
              }

    3. To test this, turn off your device location and run the app.
       You should see this pop up, press OK.

https://video.udacity-data.com/topher/2019/November/5dcc8795_image1/image1.png


Reference Documentation
ResolvableApiException
https://developers.google.com/android/reference/com/google/android/gms/common/api/ResolvableApiException.html
________________________________________________________________________________

                            5. Adding Geofences
________________________________________________________________________________

Creating a Pending Intent

  A PendingIntent is a description of an Intent and target action to
  perform with it. You will create one for the IntentService to handle
  the geofence transitions.

    1. In HuntMainActivity.kt, create a private variable above onCreate()
       called geofencePendingIntent of type PendingIntent to handle the
       geofence transitions. Connect it to the
       GeofenceTransitionsBroadcastReceiver.

    Note: The broadcast receiver is how Android apps can send or receive
          broadcast messages from the Android system and other Android apps.
          This will be explained more in the next video.

              private val geofencePendingIntent: PendingIntent by lazy {
                 val intent = Intent(this, GeofenceBroadcastReceiver::class.java)
                 intent.action = ACTION_GEOFENCE_EVENT
                 PendingIntent.getBroadcast(this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT)
              }

    Note: For more information on lazy properties (value gets computed only
          on first access), check out the Kotlin docs.


Add Geofencing Client

  A GeofencingClient is the main entry point for interacting with the
  geofencing APIs.

    1. In the onCreate() method, instantiate the geofencingClient.

              geofencingClient = LocationServices.getGeofencingClient(this)

Add geofences

    1. Copy this code into the addGeofenceForClue() method. We know it is a lot
       of code, but do not fear, we will go over the code step by step.

              private fun addGeofenceForClue() {
                 if (viewModel.geofenceIsActive()) return
                 val currentGeofenceIndex = viewModel.nextGeofenceIndex()
                 if(currentGeofenceIndex >= GeofencingConstants.NUM_LANDMARKS) {
                     removeGeofences()
                     viewModel.geofenceActivated()
                     return
                 }
                 val currentGeofenceData = GeofencingConstants.LANDMARK_DATA[currentGeofenceIndex]

                 val geofence = Geofence.Builder()
                     .setRequestId(currentGeofenceData.id)
                     .setCircularRegion(currentGeofenceData.latLong.latitude,
                         currentGeofenceData.latLong.longitude,
                         GeofencingConstants.GEOFENCE_RADIUS_IN_METERS
                     )
                     .setExpirationDuration(GeofencingConstants.GEOFENCE_EXPIRATION_IN_MILLISECONDS)
                     .setTransitionTypes(Geofence.GEOFENCE_TRANSITION_ENTER)
                     .build()

                 val geofencingRequest = GeofencingRequest.Builder()
                     .setInitialTrigger(GeofencingRequest.INITIAL_TRIGGER_ENTER)
                     .addGeofence(geofence)
                     .build()

                 geofencingClient.removeGeofences(geofencePendingIntent)?.run {
                     addOnCompleteListener {
                         geofencingClient.addGeofences(geofencingRequest, geofencePendingIntent)?.run {
                             addOnSuccessListener {
                                 Toast.makeText(this@HuntMainActivity, R.string.geofences_added,
                                     Toast.LENGTH_SHORT)
                                     .show()
                                 Log.e("Add Geofence", geofence.requestId)
                                 viewModel.geofenceActivated()
                             }
                             addOnFailureListener {
                                 Toast.makeText(this@HuntMainActivity, R.string.geofences_not_added,
                                     Toast.LENGTH_SHORT).show()
                                 if ((it.message != null)) {
                                     Log.w(TAG, it.message)
                                 }
                             }
                         }
                     }
                 }
              }

    * First, check if we have any active geofences for our treasure hunt.
      If we already do, we shouldn't add another. (After all, we only want
      them looking for one treasure at a time).

              if (viewModel.geofenceIsActive()) return

    * Find out the currentGeofenceIndex using the viewModel. If the index
      is higher than the number of landmarks we have, it means the user
      has found all the treasures. Remove geofences, call geofenceActivated
      on the viewModel, then return.

              val currentGeofenceIndex = viewModel.nextGeofenceIndex()
              if(currentGeofenceIndex >= GeofencingConstants.NUM_LANDMARKS){
                 removeGeofences()
                 viewModel.geofenceActivated()
                 return
              }

    * Once you have the index and know it is valid, get the data
      surrounding the geofence.

              val currentGeofenceData = GeofencingConstants.LANDMARK_DATA[currentGeofenceIndex]

    * Build the geofence using the geofence builder, the information in
      currentGeofenceData, like the id and the latitude and longitude.
      Set the expiration duration using the constant set in
      GeofencingConstants. Set the transition type to
      GEOFENCE_TRANSITION_ENTER. Finally, build the geofence.

              val geofence = Geofence.Builder()
                 .setRequestId(currentGeofenceData.id)
                 .setCircularRegion(currentGeofenceData.latLong.latitude,
                     currentGeofenceData.latLong.longitude,
                     GeofencingConstants.GEOFENCE_RADIUS_IN_METERS
                 )
                 .setExpirationDuration(GeofencingConstants.GEOFENCE_EXPIRATION_IN_MILLISECONDS)
                 .setTransitionTypes(Geofence.GEOFENCE_TRANSITION_ENTER)
                 .build()

    * Build the geofence request. Set the initial trigger to
      INITIAL_TRIGGER_ENTER, add the geofence you just built
      and then build.

              val geofencingRequest = GeofencingRequest.Builder()
                 .setInitialTrigger(GeofencingRequest.INITIAL_TRIGGER_ENTER)
                 .addGeofence(geofence)
                 .build()

    * Call removeGeofences() on the geofencingClient to remove any geofences
      already associated to the pending intent

              geofencingClient.removeGeofences(geofencePendingIntent)?.run {

              }

    * When removeGeofences() completes, regardless of its success or
      failure, add the new geofences.

              addOnCompleteListener {
                 geofencingClient.addGeofences(geofencingRequest, geofencePendingIntent)?.run {

                        }
              }
    * If adding the geofences is successful, let the user know through a toast
      that they were successful.

              addOnSuccessListener {
                 Toast.makeText(this@HuntMainActivity, R.string.geofences_added,
                     Toast.LENGTH_SHORT)
                     .show()
                 Log.e("Add Geofence", geofence.requestId)
                 viewModel.geofenceActivated()
              }

    * If adding the geofences fails, present a toast letting the user know that
      there was an issue in adding the geofences.

              addOnFailureListener {
                 Toast.makeText(this@HuntMainActivity, R.string.geofences_not_added,
                     Toast.LENGTH_SHORT).show()
                 if ((it.message != null)) {
                     Log.w(TAG, it.message)
                 }
              }

    2. Run the app! Your screen will now display a clue and there will be a
       toast presented that tells you that the geofence is added.

https://video.udacity-data.com/topher/2019/November/5dcc8b67_image3/image3.png

Reference Documentation
PendingIntent
https://developer.android.com/reference/android/app/PendingIntent
GeofencingClient
https://developers.google.com/android/reference/com/google/android/gms/location/GeofencingClient
________________________________________________________________________________

                            6. Update Broadcast Receiver
________________________________________________________________________________

  Your geofences are being added! However, try to navigate to the Golden
  Gate Bridge (the correct location for the default first clue).
  Nothing happens, why?

  You will need to create a Broadcast Receiver to receive the details
  about the geofence transition events. Specifically, you want to know
  when the user has entered the geofence.

  Android apps can send or receive broadcast messages from the Android system
  and other apps using Broadcast Receivers. These work in the
  publish-subscribe design pattern where broadcasts are sent out and
  apps can register to receive specific broadcasts. When the desired
  broadcasts are sent out, the apps are notified.

    1. In GeofenceBroadcastReceiver.kt, find the onReceive() function and
       copy this code into the class, we will go over the components
       in the next few steps.

              override fun onReceive(context: Context, intent: Intent) {
                 if (intent.action == ACTION_GEOFENCE_EVENT) {
                     val geofencingEvent = GeofencingEvent.fromIntent(intent)

                     if (geofencingEvent.hasError()) {
                         val errorMessage = errorMessage(context, geofencingEvent.errorCode)
                         Log.e(TAG, errorMessage)
                         return
                     }

                     if (geofencingEvent.geofenceTransition == Geofence.GEOFENCE_TRANSITION_ENTER) {
                         Log.v(TAG, context.getString(R.string.geofence_entered))
                         val fenceId = when {
                             geofencingEvent.triggeringGeofences.isNotEmpty() ->
                                 geofencingEvent.triggeringGeofences[0].requestId
                             else -> {
                                 Log.e(TAG, "No Geofence Trigger Found! Abort mission!")
                                 return
                             }
                         }
                         val foundIndex = GeofencingConstants.LANDMARK_DATA.indexOfFirst {
                             it.id == fenceId
                         }
                         if ( -1 == foundIndex ) {
                             Log.e(TAG, "Unknown Geofence: Abort Mission")
                             return
                         }
                         val notificationManager = ContextCompat.getSystemService(
                             context,
                             NotificationManager::class.java
                         ) as NotificationManager

                         notificationManager.sendGeofenceEnteredNotification(
                             context, foundIndex
                         )
                     }
                 }
              }

    * A Broadcast Receiver can receive many types of actions, but in our
      case we only care about when the geofence is entered. Check that the
      intent’s action is of type ACTION_GEOFENCE_EVENT.

              if (intent.action == ACTION_GEOFENCE_EVENT) {
              }

    * Create a variable called geofencingEvent and initialize it to
      GeofencingEvent with the intent passed in to the onReceive() method.

              val geofencingEvent = GeofencingEvent.fromIntent(intent)

    * In the case that there is an error, you will want to understand what
      went wrong. Save a variable with the error message obtained through
      the geofences error code. Log that message and return out of the method.

              if (geofencingEvent.hasError()) {
                 val errorMessage = errorMessage(context, geofencingEvent.errorCode)
                 Log.e(TAG, errorMessage)
                 return
              }

    * Check if the geofenceTransition type is ENTER.

              if (geofencingEvent.geofenceTransition == Geofence.GEOFENCE_TRANSITION_ENTER) {}

    * If the triggeringGeofences array is not empty, set the fenceID to the
      first geofence’s requestId. We would only have one geofence active at
      a time, so if the array is non-empty then there would only be one
      for us to interact with. If the array is empty, log a message and return.

              val fenceId = when {
                 geofencingEvent.triggeringGeofences.isNotEmpty() ->
                     geofencingEvent.triggeringGeofences[0].requestId
                 else -> {
                     Log.e(TAG, "No Geofence Trigger Found! Abort mission!")
                     return
                 }
              }

    * Check that the geofence is consistent with the constants listed in
      GeofenceUtil.kt. If not, print a log and return.

              val foundIndex = GeofencingConstants.LANDMARK_DATA.indexOfFirst {
                 it.id == fenceId
              }

              if ( -1 == foundIndex ) {
                 Log.e(TAG, "Unknown Geofence: Abort Mission")
                 return
              }

     * If your code has gotten this far, the user has found a valid geofence.
       Send a notification telling them the good news!

              val notificationManager = ContextCompat.getSystemService(
                 context,
                 NotificationManager::class.java
              ) as NotificationManager

              notificationManager.sendGeofenceEnteredNotification(
                 context, foundIndex
              )

    2. Try it yourself by walking into a geofence. When you enter, a
       notification should pop up.

https://video.udacity-data.com/topher/2019/November/5dcc8bbf_image8/image8.png

    Note: If you are running an emulator, the device signals that make
          geofencing possible will not work. Use an app like Google Maps
          that gets location signals to navigate to the clue location
          and this will work.

Reference Documentation
Broadcast Overview
https://developer.android.com/guide/components/broadcasts
BroadcastReceiver
https://developer.android.com/reference/android/content/BroadcastReceiver
________________________________________________________________________________

                            7. Removing Geofences
________________________________________________________________________________

  When you no longer need geofences, it is the best practice to remove them
  in order to save battery and CPU cycles to stop monitoring.

    1. In HuntMainActivity.kt, copy this code into the removeGeofences() method:

              private fun removeGeofences() {
                 if (!foregroundAndBackgroundLocationPermissionApproved()) {
                     return
                 }
                 geofencingClient.removeGeofences(geofencePendingIntent)?.run {
                     addOnSuccessListener {
                         Log.d(TAG, getString(R.string.geofences_removed))
                         Toast.makeText(applicationContext, R.string.geofences_removed, Toast.LENGTH_SHORT)
                             .show()
                     }
                     addOnFailureListener {
                         Log.d(TAG, getString(R.string.geofences_not_removed))
                     }
                 }
              }

    * Initially, check if foreground permissions have been approved, if they
      have not then return.

              if (!foregroundAndBackgroundLocationPermissionApproved()) {
                     return
                 }

    * Call removeGeofences() on the geofencingClient and pass in
      the geofencePendingIntent.

              geofencingClient.removeGeofences(geofencePendingIntent)?.run {

              }

    * Add an onSuccessListener(), update the user that the geofences were
      successfully removed through a toast.

              addOnSuccessListener {
                 Log.d(TAG, getString(R.string.geofences_removed))
                 Toast.makeText(applicationContext, R.string.geofences_removed, Toast.LENGTH_SHORT)
                     .show()
              }

    * Add an onFailureListener() where you log that the geofences weren’t
      removed.

              addOnFailureListener {
                 Log.d(TAG, getString(R.string.geofences_not_removed))
              }
________________________________________________________________________________

                            8. Navigate to winning location
________________________________________________________________________________

    1. Navigate to the winning location either by mocking location on your
       emulator or physically walking there in person! Congratulations, you
       won this lesson!

https://video.udacity-data.com/topher/2019/November/5dcc8d01_image7/image7.png
________________________________________________________________________________

                            9. Mock Location on Emulator
________________________________________________________________________________

  Since testing this lesson is dependent on walking around you most likely
  will not be walking around while making it, you will be showing you
  how to mock location on your device.

    1. On the options menu next to your emulator tap on the … at the bottom
       of the menu to open the overflow menu

    2. On the left pane, select location.

https://video.udacity-data.com/topher/2019/November/5dcc8ecc_image9/image9.png

    3. Find the latitude and longitude of a location using these instructions
      https://support.google.com/maps/answer/18539?co=GENIE.Platform%3DDesktop&hl=en
    4. Input them into the Latitude and Longitude labels.

    5. Press Send.
________________________________________________________________________________

                            10. Coding Challenge
________________________________________________________________________________

  In this lesson you only have three geofences in the app, but the
  maximum is 100! You can add landmarks to customize your hunt and
  add more to make the treasure hunt last longer. You have everything
  you need to add another geofence to make your treasure hunt longer!

    1. In strings.xml add in your custom hint and location.

              <!-- Geofence Hints -->
              <string name="lombard_street_hint">Go to the most crooked street in the City</string>
              <!-- Geofence Locations -->
              <string name="lombard_street_location">at Lombard Street</string>

    2. In GeofenceUtils.kt, you can customize the landmarks by creating
       a LandmarkDataObject with a destination ID, destination hint,
       destination location, and destination latitude and longitude.
       Add this to the LANDMARK_DATA array with your own landmark objects.

              val LANDMARK_DATA = arrayOf(
                 LandmarkDataObject(
                     “Lombard street”,
                     R.string.lombard_street_hint,
                     R.string.lombard_street_location,
                     LatLng(37.801205, -122.426752))
              )
________________________________________________________________________________

                            11. Quiz
________________________________________________________________________________

QUESTION 1 OF 3
The method to add a pin on a map object is...
addMarker



QUESTION 2 OF 3
Maps can function without location permission.
True


QUESTION 3 OF 3
The concept of responding to defined location regions.
Geofencing
________________________________________________________________________________

                            12. Summary
________________________________________________________________________________

In this lesson, you learned how to:

    * Add permissions, request permissions, check permissions and handle permissions.

    * Check device location using the settings client.

    * Add geofences using a pending intent and geofencing client.

    * Integrate a broadcast receiver to detect when a geofence is entered
      by overriding the onReceive() method.

    * Remove geofences using the geofencing client.

    * Mock locations on the emulator.

You can check out our solution code with San Francisco locations HERE.
https://github.com/udacity/android-kotlin-geo-fences

You can learn more here:
Udacity courses:

Advanced Android
https://www.udacity.com/course/advanced-android-app-development--ud855
Developing Android Apps with Kotlin
https://www.udacity.com/course/developing-android-apps-with-kotlin--ud9012


Android developer documentation:

Create and monitor geofences
https://developer.android.com/training/location/geofencing
Other resources:

Geofencing API tutorial for Android
https://www.raywenderlich.com/7372-geofencing-api-tutorial-for-android
________________________________________________________________________________
