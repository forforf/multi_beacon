# iBeacon protocol

The properties for the iBeacon protocol are all transmitted in the "Manufacturer Specific Data" field of ADV_IND (connectable, undirected).

There are four fields:

1. UUID
2. Major value (for application specific purposes)
3. Minor value (for application specific porposes)
4. Measured power

And there are also a "Flags" field present.

## Flags

The flags field specify 
- LE General Discovery
- Whether or not both BTLE and BT BR/EDR is supported (i.e. whether the device is only BTLE or both BTLE and BT classic – also known as single mode or dual mode).

The byte values are:

For single mode:

	02 01 05
	
For dual mode:

	02 01 1A
	
## UUID

The rest of the fields (UUID, Major, Minor and measured power) are sent in a Manufacturer Data field.

The prefix of the data is:

	<length> FF
	
This is followed by two bytes for company identifier (00 4C = Apple, Inc.) and two bytes, that I frankly don't know what are:

	4C 00 02 15

And then the UUID (for UUID 83256B74-78D043A4-826905F9-DC8A44BA):

	83 25 6B 74 78 D0 43 A4 82 69 05 F9 DC 8A 44 BA

## Major, Minor and measures power

Then comes two bytes (LSB) each for Major and Minor numbers and then one byte for measured power (2's complement).

If either Major or Minor are not specified, 0000 is sent.

The default value for measures power is -58 (C6).

In this case, Major is 16, Minor is 32 and measured power is -58:

	00 10 00 20 C6

## Complete data

Here is the complete data to send. The "Length" field is 26 (1A) bytes even though there are only 25 bytes of data since the "Data type" identifier (FF for "Manufacturer Data") is included: 

	02 01 1A 1A FF 4C 00 02 15 83 25 6B 74 78 D0 43 A4 82 69 05 F9 DC 8A 44 BA 00 10 00 20 C6

## BGScript example

To program a BLE112 to act as a iBeacon using BGScript, use this code:

	dim data( 30 )

	event system_boot( major, minor, patch, build, ll_version, protocol_version, hw )
	
		# Set advertisement interval to 100 to 500ms. Use all advertisement channels
		call gap_set_adv_parameters( 20, 100, 7 )
		
		#set to advertising mode
		call gap_set_mode( gap_general_discoverable, gap_undirected_connectable )
		
		# Initialize iBeacon advertisement data
		# Flags = LE General Discovery, single mode device (02 01 06)
		# UUID = 83 25 6b 74 78 d0 43 a4 82 69 05 f9 dc 8a 44 ba
		# Major = 00 10
		# Minor = 00 20
		# Measured power = -58 ($c6)
	
		# Flags
		data( 0:1) = $02
		data( 1:1) = $01
		data( 2:1) = $06
		
		# Manufacturer data
		data( 3:1) = $1a
		data( 4:1) = $ff
		
		# Preamble
		data( 5:1) = $4c
		data( 6:1) = $00
		data( 7:1) = $02
		data( 8:1) = $15
		
		# UUID
		data( 9:1) = $83
		data(10:1) = $25
		data(11:1) = $6b
		data(12:1) = $74
		data(13:1) = $78
		data(14:1) = $d0
		data(15:1) = $43
		data(16:1) = $a4
		data(17:1) = $82
		data(18:1) = $69
		data(19:1) = $05
		data(20:1) = $f9
		data(21:1) = $dc
		data(22:1) = $8a
		data(23:1) = $44
		data(24:1) = $ba
		
		# Major
		data(25:1) = $00
		data(26:1) = $10
		
		# Minor
		data(27:1) = $00
		data(28:1) = $20
		
		# Measures power
		data(29:1) = $c6
	
		# Set advertisement data
		call gap_set_adv_data(0, 30, data(0:30))
	
		#set bondable mode
		call sm_set_bondable_mode(1)
	end

	# Disconnection event listener
	event connection_disconnected(handle, result)
		# Reset connection parametesr
	    call gap_set_mode( gap_general_discoverable, gap_undirected_connectable )
		call gap_set_adv_parameters( 20, 100, 7 )
	end