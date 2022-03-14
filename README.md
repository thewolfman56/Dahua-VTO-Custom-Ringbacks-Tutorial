# Dahua-VTO-Custom-Ringbacks-Tutorial
Tutorial for using custom ringbacks with Dahua VTOs and freePBX with automatic changes for holidays, events, birthdays, time of day, etc.

Based on original information found at

[https://www.reddit.com/r/homeassistant/comments/kb7oxz/merry\_christmas\_with\_dahua\_vto\_home\_assistant\_and/](https://www.reddit.com/r/homeassistant/comments/kb7oxz/merry_christmas_with_dahua_vto_home_assistant_and/)

<h2>1. Setup freePBX</h2>

Create a pjSIP extension for your VTO in freePBX

- Go to Applications → Extensions
  - Click on Quick Create Extension

![Tutorial 1](https://user-images.githubusercontent.com/71470070/158248664-7aaa3115-bc8c-4d76-ab11-cfd7851dcded.png)

    - Type → SIP [chan\_pjsip]
    - Extension Number → The extension of your VTO (I used 8001)
    - Display Name → What you want your caller ID to show up as (I used Doorbell)
    - Outbound Caller ID → Leave this blank
    - Email Address → Leave this blank
  - Click Next

![Tutorial 2](https://user-images.githubusercontent.com/71470070/158248758-c48a7659-5f21-4100-82a5-32651e076733.png)


    - Enable Find ME/Follow Me → No
    - Create User Manager User → No preference, but usually No
    - Enable Voicemail → No
  - Click Finish
  - Click Apply Config
  - Click the Pencil Icon to Edit the newly created extension
  - Click on the Advanced Tab
    - Scroll down to Extension Options

![Tutorial 3](https://user-images.githubusercontent.com/71470070/158248832-41106be1-990e-4bf9-bca7-e25110abc7df.png)

    - Internal Auto Answer → Intercom
    - Intercom Mode → Enabled
      - These will allow you to dial the VTO directly with auto answer by the VTO from your softphone
- Create additional extensions for softphones, tablets, etc. as necessary
- Go to Applications → Ring Groups
  - Click on Add Ring Group
    - Ring-Group Number → 8999 (any number you want, but remember it)
    - Group Description → Quick description of this ring group
    - Extension List → List each extension you want to ring as a group on separate lines
    - Click Submit
    - Click Apply Config

- Go to Applications → Extensions
  - Click on Quick Create Extension

![Tutorial 4](https://user-images.githubusercontent.com/71470070/158248869-ab270a83-fd3a-4161-ab2d-86f2419946ad.png)

    - Type → Virtual
    - Extension Number → The extension of your virtual call group (I used 9001)
    - Display Name → Virtual Extension to Call Group
    - Outbound Caller ID → Leave this blank
    - Email Address → Leave this blank
  - Click Next

![Tutorial 5](https://user-images.githubusercontent.com/71470070/158248985-be41ab5d-ff1d-4cfb-ac18-4cfc6ca65742.png)

    - Enable Find ME/Follow Me → Yes
    - Create User Manager User → No preference, but usually No
    - Enable Voicemail → No
  - Click Finish
  - Click Apply Config
  - Click the Pencil Icon to Edit the newly created extension
  - Edit extension 9001
    - Go to Find-ME/Follow Me Tab
      - Follow-Me List → 8999#
      - Click Submit
      - Click Apply Config

<h2>2. Setup Dahua VTO (example for my VTO model)</h2>

- Log into VTO webpage and go to Local Settings

![Tutorial 6](https://user-images.githubusercontent.com/71470070/158249014-73e493fe-f2d4-47b8-8538-532c24bdc26a.png)


Fill in the following (and remember for later)

1. The your VTO will be calling (The virtual extension we created in freePBX)
2. Your VTOs extension

Go to Local Settings → Video &amp; Audio → Audio

![Tutorial 7](https://user-images.githubusercontent.com/71470070/158249041-2aa109fe-9029-4bca-8578-fa8070fcf317.png)

Disable all sounds except for Ringback Sound

Go to Network → SIP Server

![Tutorial 8](https://user-images.githubusercontent.com/71470070/158249088-f40f3d79-6506-41fe-85fb-f702c1aa9407.png)

- SIP Server → Disabled
- Server Type → select either Asterisk or Third-Party
- IP Address → Your freePBX Server Address
- Port → The port required for your extension registration (I used pjSIP, so my port is 5060)
- Username → the extension reserved in freePBX for your VTO
- Password → Secret found in freePBX for Extension 8001
- SIP Domain → VDP
- SIP Server Username → Leave Blank
- SIP Server Password → I used the same secret for Extension 8001 here
- Click Save

<h2>3. Additional freePBX Setup</h2>

- Go to Settings → Music on Hold
  - Click +Add Category
    - Category Name → the name of your holiday, event, etc, or your default ringback (MeowMix is the default for my example)
    - Type → Files
  - Click Submit
  - Click Apply Config

![Tutorial 9](https://user-images.githubusercontent.com/71470070/158249111-e10de899-7346-466e-901f-26b718ee9b69.png)


  - Click Edit for the new category (MeowMix)
    - Type → Files
    - Enable Random Play → If you want a mix of songs to be played, select Yes
      - I did this for Halloween and Christmas
    - Convert Upload/Files To → g722, sin, ulaw, wav

![Tutorial 10](https://user-images.githubusercontent.com/71470070/158249137-c843e595-e813-4a3f-a432-58efd1b51d12.png)

    - After selecting conversions click Browse
      - Select the music file you want to make a ringback
      - continue uploading files to the specific category by selecting Browse
    - Click Submit
    - Click OK
    - Click Apply Config
- Go to Admin → Config Edit
- Click extensions\_custom.conf (this is where all of the details come into play)
  - First, let's create some music categories for the ringback
    - Create a Label
      - [meowmix-music]
    - Create some functions
      - exten => s,1,Dial(PJSIP/8004&amp;PJSIP/8005,30,m(MeowMix))
      - exten => 9001,2,Hangup()
    - looking at the first line, here is how it is used
      - s → specifically defined extension in Asterisk
      - 1 → priority of the function for the specific function label we created
      - Dial → instruction
      - PJSIP/8004&amp;PJSIP/8005
        - these are the specif extensions that will receive the call from the VTO
        - if you want more extensions called (example a SIP extension 8006), then it would look like:
          - exten => s,1,Dial(SIP/8006,30,m(MeowMix))
        - I have not figured out the syntax for Virtual Extensions or Ring Groups, yet here.
      - 30 → the time in seconds the ringback plays
      - m(MeowMix) → the On Hold Music category we are selecting for this category
      - 9001 → the extension that will be dialed by the VTO
      - Hangup() → instructs the system to hangup after the time we selected for music to play if no answer

![Tutorial 11](https://user-images.githubusercontent.com/71470070/158249161-21b0ef63-375c-49cc-a0fc-b814fab2288c.png)

  - Now, we will instruct freePBX on what to do when the VTO calls and automate how the ringback is selected

![Tutorial 12](https://user-images.githubusercontent.com/71470070/158249197-90d81953-3c13-4654-a320-eb92037b0c57.png)

    - These instructions must be before the labels created for the music categories
    - create the label
      - [from-internal-custom]
    - Define the functions
      - exten => 9001,1,Ringing()
        - tells the system if 9001 is dialed to ring as 1st priority
      - exten => 9001,2,Answer()
        - Allows extension 9001 to be answered as the 2nd priority
      - exten => 9001,3,GotoIfTime(00:00-23:59,\*,\*,oct?halloween-music,s,1)
        - Priority is important here as the highest priority (lowest number) match will take priority over any other match
        - We use the GotoIfTime function to automate the parameters of when a specific music On Hold Category will be played category
          - The syntax of GotoIfTime for our example is:
          - GotoIfTime(<time range>,<days of week range>,<days of month range>,<months range>?[Label if true],[extension],[priority])
            - So our example will play Halloween Music any time, any day, and on any date in October
            - See line 6 in the photo for an example that is a little more defined around St. Patrick's Day
      - The last line for this label, use Goto for the default ringtone music Group you want to use
 - Click Save and then Apply Config (may have to reboot freePBX to take effect)
