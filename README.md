# Chronos-Wearable-Disk
My attempt to decipher the UART commands to customization and a possible new app.

Official Website: https://wearchronos.com/

Figured Out:

	Light:
		bRGB
		 000 - Black
		 111 - White
		 010 - Green
		 ...

	Vibration:
		a (acivate?) - Defined Vibration and Flash

	Reset?:
		R - Caused disconnect and loss of preferences.

	???:
		G - Returns -1 and 5 alternating seemingly randomly
		S - (Status?) Returned "0, 0 Stationary" and sometimes "14, 0 Stationary"


Working Out:

	Sent G, Got -1 and 5
	Send S, Got "0, 0 Stationary"
	Sent Delta Triangle, Lights Turned Full -------+
	Sent Square Root Symbol, Lights Turned Full    +----> Unicode Characters = 111
	Sending to Unknown Service:

		Assumption = Vibrate 3 Times and Flash White
		0x91-11-61-01-83-1e-ff-2f-03-1e-04-2f-02-1e-02-2f-02-20-03 // Set Vibration and Light Preference?
		0x90-0a-40-01-31-82-31-82-31-00-00-00 // Save to Device?
		0x95-04-61-01-40-01 // Activation


		Assumption = Vibrate 2 Times and Flash White
		0x91 11 61 01 82 1e ff 2f 06 1e 04 2f 02 1e 02 2f 02 20 06
		0x90 0a 40 01 0e 8f 0e 00 00 00 00 00
		0x95 04 61 01 40 01
		            ^  ^  ^
		            |  |  +---- 00 = No Vibrate
		            |  +---- If not 40 then No Vibrate
		            +---- 01 = Flash White 2 Times (As Defined Previously?)
		            +---- 00 = Flash Yellow 3 Times (Unknown Notification?)


		Assumption = Vibrate 1 Time and Flash White
		0x91 11 61 01 81 1e ff 2f 18 1e 04 2f 02 1e 02 2f 02 20 18
		0x95 04 61 01 40 01
		0x90 0a 40 01 2f 2f 00 00 00 00 00 00 -- Happened out of order. Confirmation of Save?

		Deduction:
			0x91 = Create Pattern
			0x90 = Save Pattern
			0x95 = Activate Command with Separate Triggers for Vibration and Lights

		Pattern Creation:
			0x91-11-61-01-83-1e-ff-2f-03-1e-04-2f-02-1e-02-2f-02-20-03 = 1 Times 3x1=1
			0x91 11 61 01 82 1e ff 2f 06 1e 04 2f 02 1e 02 2f 02 20 06 = 2 Times 3x2=6
			0x91 11 61 01 81 1e ff 2f 18 1e 04 2f 02 1e 02 2f 02 20 18 = 3 Times 3x6=18
			0x91 11 61 01 80 1e ff 2f 24 1e 04 2f 02 1e 02 2f 02 20 24 = 3 Times 4x6=24 ?
			               ^           ^                             ^
			               |           |                             |
			               |           |                             +--------+
			               |           |                                      |
			               |           +---- Times / 3 to Repeat Vib/Flash? --+
			               +---- Predefined Mode of Vibration?

		Pattern Saving:
			0x90-0a-40-01-31-82-31-82-31-00-00-00
			0x90 0a 40 01 0e 8f 0e 00 00 00 00 00
			0x90 0a 40 01 2f 2f 00 00 00 00 00 00

		Activation:
			0x95-04-61-00-40-00 = Show Error Flash with No Vibration
			0x95-04-61-00-40-01 = Vibrate without Flashing
			0x95-04-61-01-40-00 = Flash without Vibrating
			0x95-04-61-01-40-01 = Flash and Vibrate
			            ^     ^
			            |     |
			            |     +---- Vibrate (0/1)
			            |
			            +---- Flash (0/1)

		

		0x91-11-61-01-82-1e-ff-2f-06-1e-04-2f-02-1e-02-2f-02-20-06 --- ?
		0x91-11-61-01-83-1c-ff-2f-03-1c-04-2f-02-1c-02-2f-02-20-03 = Disable Flash on Vibrate? (Also Vibrates)
                    ^              ^
                    |              +---- When set to 00 nothing happens.
                    |
                    |
                    +---- When set to 00 nothing happens.
                    0x91-04-2f-02-1c-02-2f-02-20-03 = Nothing happens
		



App Control:
	Disabled Incoming Call, Media Playback, Find Phone
	Enabled Incoming Call, Media Playback, Find Phone

	Sent 9c00 -- Disable Incoming Call
	Sent 9d00 - Disable Media Playback?
	Sent b00101 -- Enable Incoming Call?
	Sent b20101 -- Enable Media Playback?
	Sent b30101 -- Enable Find Phone?


	On Media Playback Gesture: Received 0x20-20 (" ")
	On Find my Phone Gesture: Received 0x28-00
	On Camera Gesture: Received: 0x20-02
	On unknown gesture?: Received: 0x21-00

	Sent Read Request (0x0a) with handle 0x0039 and received 0x0b of value 0x20-00