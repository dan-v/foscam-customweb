#!/bin/bash
# by neXus April 2011
# v1.1
# Changelog
# 26/04/2011 - Fixe size issue for webui, escape var to allow white space in path

function Get_Bytes
{
xxd -ps -s $2 -l 4 -g 16 -c 4 "$1" 
}

function Get_LowHigh
{
 echo $1 | sed -r 's/([0-9a-f][0-9a-f])+([0-9a-f][0-9a-f])+([0-9a-f][0-9a-f])+([0-9a-f][0-9a-f])/\4\3\2\1/' 
}


function Get_Version
{
	Get_Bytes "$1" 12 | awk '{printf("%s.%s.%s.%s",strtonum("0x"substr($1,1,2)),strtonum("0x"substr($1,3,2)),strtonum("0x"substr($1,5,2)),strtonum("0x"substr($1,7,2)))}' 
}


function Get_Hash
{
	xxd -ps -s $2 -c 1 "$1" | awk '{sum+=strtonum("0x"$1)} END{printf "%08x",sum}'
}


function WebUI_Unpack
{
	SEEK=16
	END=`stat --printf %s "$1"`	
	BASE=$1_extracted	
	echo "Extracting to $BASE"
	while [ $SEEK -ne $END ]; do
	
		#Get size of filename
		N_SIZE=`echo $(Get_LowHigh $(Get_Bytes "$1" $SEEK)) | awk '{print strtonum("0x"$1)}'`
		let SEEK+=4
		
		#Get file name
		NAME=`xxd -s $SEEK -l $N_SIZE "$1" |xxd -r`
		let SEEK+=$N_SIZE
		
		# Test for bit. if 01 its a file else folder
		STR_BIT=`xxd -ps -s $SEEK -l 1 "$1"`
		let SEEK+=1
		if [ $STR_BIT = "00" ]; then
			echo ">Folder $NAME"
			mkdir -p $BASE$NAME

		else

			#Get file size
			F_SIZE=`echo $(Get_LowHigh $(Get_Bytes "$1" $SEEK)) | awk '{print strtonum("0x"$1)}'`
			let SEEK+=4	
			echo ">File $NAME ($F_SIZE)"
		
			# Check file path
			[  ! -d $BASE`dirname $NAME` ] && mkdir -p $BASE`dirname $NAME`
			# Dump the file
			dd if="$1" of=$BASE$NAME bs=1 skip=$SEEK count=$F_SIZE > /dev/null 2>&1
			let SEEK+=$F_SIZE
		fi
	
	done


}

function WebUI_Pack
{
	
	# create binary package
	read -p "Type the name of bin output: " BIN_FILE
	read -p "Type the version to set : (xx.xx.xx.xx) " VERS
	[ -z "$BIN_FILE" ] && echo "Please set a name" && exit 1
	OUTPUT="`pwd`/$BIN_FILE"	
	BASE="$1"
	[ -f "$OUTPUT" ] && rm "$OUTPUT"
	DATA="$OUTPUT.data"
	HEADER="$OUTPUT.header"
	# find files and folder
	cd "$1"	
	echo "Remove existing headers"
	find -iname "*.header" -delete
	echo ">Adding version to output"
       	echo "00:"`echo $VERS |awk -F. '{printf("%02x%02x%02x%02x",$1,$2,$3,$4)}'` |xxd -r >"$DATA"
	
	for item in `find | sed '/^.$/d' `; do
	# debug for item in `cat files.list | sed '/^$/d' |sed 's/\//\.\//'`;do
	echo "Packing -> $item"

		NAME=`echo $item |sed 's/^\.//g'`
		N_SIZE=$(Get_LowHigh `echo -n $NAME | wc -m | awk '{printf("%08x",$1)}'`)

	if [ -f $item ]; then
	
		F_SIZE=$(Get_LowHigh `stat --printf %s $item | awk '{printf("%08x",$1)}'`)

		# create the header file we need to process by step
		RAW_HEX=$N_SIZE`echo -n $NAME | xxd -ps`"01"$F_SIZE

	else

		# create the header file for folder
		RAW_HEX=$N_SIZE`echo -n $NAME | xxd -ps`"00"
	fi

		CAR=`echo $RAW_HEX -n |wc -m`
		POS=0
		count=0
		while [ $POS -lt $CAR  ]; do
        		PART=`echo ${RAW_HEX:$POS:16}`
		        let POS+=16
		        echo "$count: $PART" | xxd -r >> "$item.header"
			let count++
		done
		# create the output file with header and file or header only if folder
		cat "$item.header" >> "$DATA"
		[ -f "$item" ] && dd if="$item" of="$DATA" conv=notrunc oflag=append > /dev/null 2>&1 
	
	done	
	# create header with size / crc / version
	echo ">Calculating checksum"
	CHKS=$(Get_LowHigh $(Get_Hash "$DATA" 0))
	echo ">Get Size"
	SIZE=$(Get_LowHigh `stat --printf %s "$DATA" | awk '{printf("%08x",$1+12)}'`)
	echo ">Write Header"
	echo -n "0: BD9A0C44 $CHKS $SIZE"  | xxd -r > "$HEADER"
	cat "$HEADER" "$DATA" > "$OUTPUT"
	echo ">Check Integrity :"
	echo -e "\t->Version : $(Get_Version "$OUTPUT")"
	sum32=$(Get_LowHigh $(Get_Bytes "$OUTPUT" 4))
        calc_sum32=$(Get_Hash "$OUTPUT" 12)
	[ ! "$sum32" = "$calc_sum32" ] && echo " !!!! checksum mismatch ! File Corrupted" && exit 1
	echo -e "\t->Checksum verified ($sum32)"
	echo
	echo "Firmware successfully packed to $OUTPUT"
	find -iname "*.header" -delete
}

# main actions

function webui
{

if [ -f "$1" ]; then

	echo "Checking for file : $1"
	echo ">Version  : " $(Get_Version "$1")
	sum32=$(Get_LowHigh $(Get_Bytes "$1" 4)) 
	calc_sum32=$(Get_Hash "$1" 12)
	[ ! "$sum32" = "$calc_sum32" ] && echo "checksum mismatch ! File Corrupted" && exit 1 
	echo ">Checksum : $sum32"
	echo
	read -p "> Unpack ?(y/n)" ANS
	[ ! "$ANS" = "y" ] && exit 0
	echo "Unpacking Files..."
	WebUI_Unpack $1


else
	echo "Packing a webui"
	WebUI_Pack "$1"

fi

}

function firmware
{
if [ -f "$1" ]; then
	echo "Unpacking firmware $1"
	LINUX_SIZE=`echo $(Get_LowHigh $(Get_Bytes "$1" 12)) | awk '{print strtonum("0x"$1)}'`
	ROOTFS_SIZE=`echo $(Get_LowHigh $(Get_Bytes "$1" 16)) | awk '{print strtonum("0x"$1)}'`
	mkdir -p "$1"_extracted
	echo ">Extracting linux.bin ($LINUX_SIZE)"
	dd if="$1" of="$1_extracted"/linux.bin bs=1 skip=20 count=$LINUX_SIZE > /dev/null 2>&1
	echo ">Extracting rootfs.img ($ROOTFS_SIZE)"
	dd if="$1" of="$1_extracted"/rootfs.img bs=1 skip=$((20+$LINUX_SIZE)) count=$ROOTFS_SIZE > /dev/null 2>&1
	echo "Done extracting firmware"
	
else
	echo "Packing firmware"
	echo ">Looking for file to pack in $1"
	[ ! -f "$1/linux.bin" -a ! -f "$1/rootfs.img" ] && echo "Missing files !!" && exit 1
	LINUX_SIZE=`stat --printf %s $1/linux.bin`
	echo -e "\t linux.bin ($LINUX_SIZE)..OK"
	ROOTFS_SIZE=`stat --printf %s $1/rootfs.img`
	echo -e "\t rom1fs ($ROOTFS_SIZE)..OK"
	echo
	read -p ">Give a name to your firmware : " NAME
	LINUX_SIZE=$(Get_LowHigh `echo -n $LINUX_SIZE | awk '{printf("%08x",$1)}'`)
	ROOTFS_SIZE=$(Get_LowHigh `echo -n $ROOTFS_SIZE | awk '{printf("%08x",$1)}'`)
 	[ -f "$NAME" ] && rm "$NAME"
	echo ">Generate header"
	HEADER1="0000000: 424E4547 01000000 01000000 $LINUX_SIZE"
	HEADER2="0000010: $ROOTFS_SIZE"
	echo $HEADER1 |xxd -r > "$NAME.header"
	echo $HEADER2 |xxd -r >>"$NAME.header"
	echo ">Generate firmware"
	cat "$NAME.header" "$1/linux.bin" "$1/rootfs.img" > "$NAME"
	rm "$NAME.header"
	echo "Done genrating $NAME"
fi
}


function show_usage
{
cat <<EOT
	.:: Foscam Resource ToolKit ::.

	$0 --webui|--firm file|folder

	You can use this piece of script to :
		- Pack and unpack webui resource
		- Pack and unpack firmware resource

	Usage :
	
	Unpack webui :		$0 --webui file.bin

	Pack webui :		$0 --webui webui_folder

	Unpack firmware :	$0 --firm file.bin

	Pack firmware :		$0 --firm firmware_folder

	(note the firmware folder must contain rootfs.img and linux.bin files)

EOT

}

# main programm

[ $# -ne 2 ] && show_usage && exit 0

if [ $1 = "--webui" ]; then
	webui "$2"
elif [ $1 = "--firm" ]; then
	firmware "$2"
else
	show_usage
fi








