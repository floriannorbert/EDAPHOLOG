;******************************************************************************

	LIST P=18F14K22		;directive to define processor
	#include <P18F14K22.INC>	;processor specific variable definitions

;******************************************************************************
;Configuration bits
;Microchip has changed the format for defining the configuration bits, please 
;see the .inc file for futher details on notation.  Below are a few examples.



;   Oscillator Selection:
    CONFIG	FOSC = IRC
    CONFIG	STVREN = ON
    CONFIG	WDTEN = OFF
    CONFIG	PWRTEN = ON
    CONFIG	PLLEN = OFF
    CONFIG	PCLKEN = OFF
    CONFIG	MCLRE = OFF
    CONFIG	LVP = OFF
    CONFIG	BBSIZ = OFF	;1Kword Boot mem.
    CONFIG	XINST = OFF
    CONFIG	IESO = OFF
    CONFIG	BOREN = OFF
    CONFIG	FCMEN = OFF
    CONFIG	HFOFST = ON
    

;******************************************************************************
	radix	dec
#define	_C	STATUS,C,A
#define	_Z	STATUS,Z,A

debug	equ	1	;uncomment for conditional compilation

UnicID	equ	0x0101	;next ID, should look as decimal if bytewise printed
			;0x0200, 0x0300, ..., 0x0900, 0x1000, 0x1100, ..., 
			;0x9900, 0x0001, 0x0101, 0x0201, 0x0301, ...
EdaphoVersion equ	0x0207

#define		FastDebug	;Comment out for normal operation !!!
#define	lockout

MAXRECORDS	equ	2 ;.30
MAXREC2		equ	2 ;.10
MAXPERHOUR	equ	.12
	ifdef	FastDebug
SIMULCOUNT	equ	.2
TXTC		equ	.3	;minutes
	else
SIMULCOUNT	equ	.10
TXTC		equ	.60 * .8
	endif
REEDTC		equ	.20
RECLEN		equ	.8
SDI	equ	4 ;RB4 green LED
SCK	equ	6 ;RB6 red LED
RFPWR	equ	7 ;RC7 RFtx PWR, 1 sec toggle for debug YELLOW
MPWR	equ	5 ;RC5 HUMID, Ux power
RNG	equ	4 ;RC4 HUMID range switch
UXCHS	equ	4 * 4 ;RC0/AN4, 2 bits shift left in ADCON0
THMCHS	equ	5 * 4 ;RC1/AN5, temperature
BATTCHS	equ	6 * 4 ;RC2/AN6, Ubattery/3
OPTIC2	equ	7 * 4 ;RC3/AN7, 2 bits shift left in ADCON0
OPTICDC	equ	8 * 4 ;RC6/AN8
CHSMASK	equ	0x3c ;CHS bits in ADCON0

#define	GREENLED	LATB,SCK,A  ;yellow on in sensing
#define	REDLED		LATB,SDI,A  ;RED on while lockout

;flags bits
SECCHG	equ	0
MINCHG	equ	1
HRCHG	equ	2
RFDONE	equ	3
SWRFON	equ	4	;send coll. and measurement values in separate blocks
MEASST	equ	5	;start A/D measurements
MEASRDY	equ	6	;A/D measurements ready
WORM	equ	7

;flags2 bits
MEASPRE	equ	0
WORMLOCK equ	1
WORMING	equ	2	;INT2 irq started worm detection
TEST1	equ	3	;marks 1. cycle of test TX for REED request
MEASING	equ	4	;A/D measurements pending
THMRDY	equ	5
BATTRDY	equ	6
UXRDY	equ	7

;flags3	bits
STARTED	equ	0
WBUF2	equ	1
RSRC1	equ	2
RSRCRDY	equ	3
SIMTX	equ	4
TIMESET	equ	5
RESTART	equ	7

;humflags bits
HUMRNG	equ	0
HUMSTAB	equ	1
HUMING	equ	2
HUMINI	equ	3	;Humidity measurement 1. cycle
HUMID	equ	4	;Humidity measurement cycle ready
HUMSTART equ	5
HUMOVF	equ	6
HUMADRDY equ	7	;HUMID range: 1 if large C used

;txflags bits
NotUsed	equ	0	
STARTTX	equ	1
TXGO	equ	2
TXDOWN	equ	3
NOWORM	equ	4
RFREED equ	5	;RF modul reed on
RFCYCLE equ	6

		CBLOCK	0x00
		flags
		flags2
		flags3
		temp
		tempu
		humflags
		txflags
		wormctrl	;from here goes to ext. eeprom
		wormctrh
		secctr
		minctr
		hrctr
		dayctr
		tickctrl
		tickctrh
		tickctru	
		wormlen
		pulsestartl
		pulsestarth
		pulseendl
		pulseendh
		pulselenl
		pulselenh
		delayctr
		resil
		resih
		resiu
		huml
		humh
		humu
		humltx
		humhtx
		humutx
		thmltx
		thmhtx
		batttx
		uxltx
		uxhtx
		opdcltx
		opdchtx
		integctr
		integctrh
		opdcl
		opdch
		integ2l
		integ2h
		integ2u
		noise1ctrl
		noise1ctrh
		addelay
		thml
		thmh
		battl
		batth
		uxl
		uxh
		txctr
		meastc	;minutes between measurements
		measctr
		rscmd
		wasting
		recctr
		rxctr
		txtimerl	;timer between transmissions
		txtimerh
		flaghour
		lastctrl
		lastctrh
		lastday
		lasthour
		lastmin
		lastsec
		lastbatt
		lastthml
		lastthmh
		lasthuml
		lasthumh
		lasthumu
		tmr2ctr
		txrecctr
		rxid
		testvall
		testvalh
		rftimeout
		simctr
		simtmr
		thmidx
		rfctr
		limctr
		lastrx
		txcode
		ENDC
		
		cblock	0x60
		fsr1savel
		fsr1saveh
		fsr2savel
		fsr2saveh
		thmbuf
		ENDC

txbuf		equ	80h
wormbuf2	equ	0a0h
wormbuf		equ	0x100
;wormbuf record structure:
; 0. byte:	Colembola size
; 1. byte:	Days
; 2. byte:	Hours + flags on bit 6..7
; 3. byte:	Minutes
; 4. byte:	Seconds
; flags:	bit7 marks special record:
;		battery at 0. byte, 1-4 bytes timestemp
; 		5-6. bytes: temperature
;		7-9. bytes: humidity
;******************************************************************************
;EEPROM data
; Data to be programmed into the Data EEPROM is defined here

;		ORG	0xf00000

;		DE	"Test Data",0,1,2,3,4,5

;******************************************************************************
;Reset vector
	ORG	0x0000
	goto	Main		;go to start of main code
;******************************************************************************
	ORG	0x0004
pUnicID	dw	UnicID
	ORG	0x0008
HighInt:
	ifndef	__DEBUG
	btfss	INTCON,INT0IE,A
	bra	tfnextirq
	btfsc	INTCON,INT0IF,A
	bra	int0irq		;humid
	endif	;__DEBUG
tfnextirq
	btfss	INTCON,TMR0IE,A
	bra	tfadirq
	btfsc	INTCON,TMR0IF,A
	bra	t0irq
tfadirq
	btfsc	PIR1,ADIF,A
	bra	adirq
tfint2
	btfss	INTCON3,INT2IE,A
	bra	tft1irq
	btfsc	INTCON3,INT2IF,A
	bra	int2irq		;opto 1
tft1irq
	btfsc	PIR1,TMR1IF,A
	bra	t1irq		;realtime clock 32768 -> 1 Hz
	btfss	PIE1,RCIE,A
	bra	tftxirq
	btfsc	PIR1,RCIF,A
	bra	rsrcirq
tftxirq
	btfss	PIE1,TXIE,A
	bra	tft2irq
	btfsc	PIR1,TXIF,A
	bra	txirq
tft2irq
	btfss	PIE1,TMR2IE,A
	bra	popret
	btfsc	PIR1,TMR2IF,A
	bra	t2irq
popret
	retfie	FAST
adirq
	bcf	PIR1,ADIF,A
	movf	PCL,W,A
	rrcf	ADCON0,W,A
	andlw	0x1e	;CH0 to 15
	addwf	PCL,F,A	;jump to A/D handler according to CH nr.
	bra	humirq	;ch0
	bra	popret	;ch1
	bra	popret	;ch2
	bra	popret	;ch3
	bra	uxirq	;ch4: Ux channel
	bra	thmirq	;ch5 temperature sensor
	bra	battirq	;ch6 battery
	bra	opt2irq	;ch7 worm noise
	bra	opdcirq	;ch8 optic opamp dc output
	bra	popret	;ch9
	bra	popret	;ch10
	bra	popret	;ch11
	bra	popret	;ch12
	bra	popret	;ch13
	bra	popret	;ch14
	bra	popret	;ch15
humirq
	tstfsz	addelay,A
	bra	humaddelay
	movf	ADRESH,W,A
	addwf	huml,F,A
	btfsc	_C
	incf	humh,F,A
	cpfslt	resil,A
	movwf	resil,A
	cpfsgt	resih,A
	movwf	resih,A
	decfsz	integctr,F,A
	bra	popret
	bsf	ADCON2,ADFM,A
	clrf	ADCON0,A
	bcf	ANSEL,ANS0,A	;enable INT0 input
	bsf	humflags,HUMADRDY,A
	bra	popret
humaddelay
	decf	addelay,F,A
	bra	popret

thmirq
	tstfsz	addelay,A
	bra	chdelay
	movf	ADRESL,W,A
	addwf	thml,F,A
	movf	ADRESH,W,A
	addwfc	thmh,F,A
	decfsz	integctr,F,A
	bra	popret
	clrf	ADCON0,A
	tstfsz	thmidx,A
	bra	thmirb
	movlw	.64
	movwf	integctr,A
	movlw	OPTICDC + 1
	movwf	ADCON0,A
	clrf	opdcl,A
	clrf	opdch,A
	movff	thml,thmltx
	movff	thmh,thmhtx
	bra	popret
thmirb
	bsf	flags2,THMRDY,A
	bra	popret
opdcirq
	tstfsz	addelay,A
	bra	chdelay
	movf	ADRESL,W,A
	addwf	opdcl,F,A
	movf	ADRESH,W,A
	addwfc	opdch,F,A
	decfsz	integctr,F,A
	bra	popret
	clrf	ADCON0,A
	movlw	.64
	movwf	integctr,A
	movlw	UXCHS + 1
	movwf	ADCON0,A
	clrf	uxl,A
	clrf	uxh,A
	movff	opdcl,opdcltx
	movff	opdch,opdchtx
	bra	popret
uxirq
	tstfsz	addelay,A
	bra	chdelay
	movf	ADRESL,W,A
	addwf	uxl,F,A
	movf	ADRESH,W,A
	addwfc	uxh,F,A
	decfsz	integctr,F,A
	bra	popret
	clrf	ADCON0,A
	bcf	LATC,MPWR,A
	movlw	.64
	movwf	integctr,A
	movlw	BATTCHS + 1
	movwf	ADCON0,A
	clrf	battl,A
	clrf	batth,A
	movff	uxl,uxltx
	movff	uxh,uxhtx
	bra	popret
battirq
	tstfsz	addelay,A
	bra	chdelay
	movf	ADRESL,W,A
	addwf	battl,F,A
	movf	ADRESH,W,A
	addwfc	batth,F,A
	decfsz	integctr,F,A
	bra	popret
	clrf	ADCON0,A
	rlcf	battl,W,A	;to restore: add 256, multiply by 3, divide by 512
	rlcf	batth,W,A	;and multiply by 3 (x+256)*450/256
	movwf	batttx,A
	bcf	flags2,MEASING,A
	bsf	flags,MEASRDY,A
	bra	popret

opt2irq
	tstfsz	addelay,A
	bra	chdelay
	btfss	ADRESH,1,A
	bra	opt2neg
	movf	ADRESL,W,A
	addwf	integ2l,F,A
	movf	ADRESH,W,A
	bcf	WREG,1,A
	addwfc	integ2h,F,A
	btfsc	_C
	incf	integ2u,F,A
	bra	opt2irq2
opt2neg
	comf	ADRESL,W,A
	addwf	integ2l,F,A
	movlw	1
	xorwf	ADRESH,W,A
	addwfc	integ2h,F,A
	btfsc	_C
	incf	integ2u,F,A
opt2irq2
	dcfsnz	integctr,F,A
	decfsz	integctrh,F,A
	bra	popret
	clrf	ADCON0,A
	bra	popret
chdelay
	decf	addelay,F,A
	bra	popret
int0irq			;humidity measurement
	bcf	INTCON,INT0IF,A
	bsf	T1CON,RD16,A
	btfss	humflags,HUMINI,A
	bra	int0irq2
	movf	TMR1L,W,A	;30.52us resolution
	movwf	tickctrl,A
;	movwf	testvall,A
	movf	TMR1H,W,A
	movwf	tickctrh,A
;	movwf	testvalh,A
	bcf	T1CON,RD16,A
	clrf	tickctru,A
	btfss	tickctrh,7,A
	decf	tickctru,F,A
	btfsc	humflags,HUMSTAB,A
	bra	int0irq3
	bcf	humflags,HUMINI,A
	bra	popret
int0irq3
	bcf	humflags,HUMSTAB,A
	bra	popret
int0irq2
	movf	TMR1L,W,A	;30.52us resolution
	movwf	resil,A
	movff	TMR1H,resih
	bcf	T1CON,RD16,A
	btfss	resih,7,A
	incf	tickctru,F,A
	bcf	tickctrh,7,A
	bcf	_C
	rrcf	tickctru,W,A
	movwf	resiu,A
	bsf	resih,7,A
	btfss	_C
	bcf	resih,7,A
	movf	tickctrl,W,A
	subwf	resil,F,A
	movf	tickctrh,W,A
	subwfb	resih,F,A
	movlw	0
	subwfb	resiu,F,A
	bsf	humflags,HUMID,A
	bcf	INTCON,INT0IE,A
	movf	resiu,W,A
	btfsc	_Z
	bra	popret
	setf	resil,A
	setf	resih,A
	bra	popret
int2irq				;1. optosensor
	bcf	INTCON3,INT2IF,A
	btfsc	flags2,WORMLOCK,A
	bra	int2locked
	bsf	GREENLED
	btfsc	INTCON2,INTEDG2,A
	bra	int2rise
	bsf	INTCON2,INTEDG2,A	;set rising edge irq
	btfsc	flags2,WORMING,A	;skip if start sensing
	bra	int2irq3	;already sensed
int2irq1
	bsf	OSCCON,IRCF2,A	;start sensing, swithc to 2MHz clock
	bcf	INTCON,INT0IE,A	;disable Humid measurement
	bsf	flags2,WORMING,A
	movlw	OPTIC2 + 1	;start A/D, set input to opto ampl.
	movwf	ADCON0,A
	movlw	.2
	movwf	addelay,A
	movlw	.2	;51.2 ms integr. time
	movwf	integctrh,A
	clrf	integctr,A
	clrf	integ2l,A
	clrf	integ2h,A
	clrf	integ2u,A
int2irq2  ;start sensing
	movlw	high (-.12625)	;~400ms, new pulse start
	movwf	TMR0H,A
	movwf	pulsestarth,A
	movwf	pulseendh,A
	movlw	low (-.12625)
	movwf	TMR0L,A
	movwf	pulsestartl,A
	movwf	pulseendl,A
	bra	popret	
int2rise
	bcf	INTCON2,INTEDG2,A	;set falling edge irq
	btfss	flags2,WORMING,A
	bra	int2irq1
int2irq3	;continue sensing
	movff	TMR0L,pulseendl
	movff	TMR0H,pulseendh
	bra	popret
int2locked
	bra	popret
t0irqunlock
	btfsc	txflags,RFCYCLE,A
	bra	popret
	bcf	flags2,WORMLOCK,A
	bcf	REDLED
	btfss	PORTA,1,A
	bsf	INTCON2,INTEDG1,A
	btfsc	PORTA,1,A	
	bcf	INTCON2,INTEDG1,A
	btfss	PORTA,2,A
	bsf	INTCON2,INTEDG2,A
	btfsc	PORTA,2,A	
	bcf	INTCON2,INTEDG2,A
	bra	popret
t0irq
	bcf	INTCON,TMR0IF,A	;~ 0.48Hz, started in int2irq 
	btfsc	flags2,WORMLOCK,A
	bra	t0irqunlock
	btfss	flags2,WORMING,A
	bra	popret		;does nothing if not worming
	bcf	flags2,WORMING,A
	movf	pulsestartl,W,A
	subwf	pulseendl,W,A
	movwf	pulselenl,A
	movf	pulsestarth,W,A
	subwfb	pulseendh,W,A
	movwf	pulselenh,A
	bsf	flags,WORM,A
	bsf	flags2,WORMLOCK,A
	bsf	REDLED
	btfsc	flags2,MEASING,A
	bsf	flags2,MEASPRE,A  ;restart measurements if pending
	bra	popret
t1irq0
	bsf	TMR1H,6,A
	bsf	TMR1H,5,A
	decfsz	secctr,F,A
	bra	popret
	bsf	flags3,STARTED,A
	bra	popret
t1irq				;base clock timer
	bcf	PIR1,TMR1IF,A
	bsf	TMR1H,7,A	;set for 1 sec. irq
	btfss	flags3,STARTED,A
	bra	t1irq0
	incf	secctr,F,A
	incf	tickctru,F,A
t1irq21
;	bra	t1irq22

	btfss	OSCCON,IRCF2,A
	bra	t1irq22
	btfsc	flags2,MEASING,A	;2MHz clock
	bra	t1irq22
	btfsc	txflags,RFCYCLE,A	;no measing
	bra	t1irq22
	decfsz	wasting,F,A		;no rf 
	bra	t1irq22
	bcf	LATC,RFPWR,A
	bcf	TXSTA,TXEN,A
	bcf	RCSTA,CREN,A
	bcf	RCSTA,SPEN,A
	bsf	TRISC,RFPWR,A
	bcf	LATB,TX,A
	bcf	OSCCON,IRCF2,A
t1irq22
	bsf	flags,SECCHG,A
	btfss	txflags,RFCYCLE,A
	bra	t1irq32
	decfsz	rftimeout,F,A
	bra	t1irq32
rffreese		;RF cycle timed out
	movlw	2
	movwf	rftimeout,A
	bcf	TXSTA,TXEN,A
	bcf	RCSTA,CREN,A
	bcf	RCSTA,SPEN,A
	bcf	LATC,RFPWR,A
	bcf	LATB,TX,A
	clrf	TMR2,A		;init tx-down timer
	bcf	PIR1,TMR2IF,A
	bsf	PIE1,TMR2IE,A
	movlw	.100
	movwf	tmr2ctr,A
t1irq32
	movlw	.60
	cpfseq	secctr,A
	bra	popret
	clrf	secctr,A
	incf	minctr,F,A
	bsf	flags,MINCHG,A
	;txtimer here with minutes base
	decfsz	measctr,F,A
	bra	t1irq3
	movff	meastc,measctr
	bsf	flags2,MEASPRE,A
;	bsf	humflags,HUMSTART,A
t1irq3
	
	movlw	1
	subwf	txtimerl,F,A
	movlw	0
	subwfb	txtimerh,F,A
	movf	txtimerl,W,A
	iorwf	txtimerh,W,A
	bnz	t1irq33
	movlw	low (TXTC)
	movwf	txtimerl,A
	movlw	high (TXTC)
	movwf	txtimerh,A
	bsf	flags,SWRFON,A
t1irq33
	movlw	.60
	cpfseq	minctr,A
	bra	popret
	clrf	minctr,A
	incf	hrctr,F,A
	clrf	limctr,A
	bsf	flags,HRCHG,A
t1irq26
	movlw	.24
	cpfseq	hrctr,A
	bra	popret
	clrf	hrctr,A
	incf	dayctr,F,A
	bra	popret
rsrcerr
	movf	RCREG,W,A
	bsf	OSCCON,IRCF2,A
	bcf	RCSTA,CREN,A
	clrf	rxctr,A
	movlw	3
	movwf	wasting,A
	bsf	RCSTA,CREN,A
	btfss	PIR1,RCIF,A
	bra	popret
rsrcirq
	movf	RCSTA,W,A
	andlw	0x06
	btfss	_Z
	bra	rsrcerr
	movff	RCREG,rscmd
	bsf	flags3,RSRCRDY,A
	bra	popret
txirq
	movff	POSTINC1,TXREG
	decfsz	txctr,F,A
	bra	popret
txirqdone
	bcf	PIE1,TXIE,A
	btfss	flags,RFDONE,A
	bra	txgoon
	bra	popret
	clrf	TMR2,A		;init tx-down timer
	bcf	PIR1,TMR2IF,A
	bsf	PIE1,TMR2IE,A
	movlw	.2
	movwf	tmr2ctr,A
	bra	popret
txgoon
	bra	popret
	btfsc	txflags,RFREED,A
	bra	popret
	bsf	txflags,TXGO,A
	bra	popret
t2irq			;TX-down timer 
	bcf	PIR1,TMR2IF,A
	decfsz	tmr2ctr,F,A
	bra	popret
	bcf	PIE1,TMR2IE,A
	bsf	TRISC,RFPWR,A	;RFCYCLE end, RFPWR pin input to sense reed activation
	bcf	txflags,RFCYCLE,A
	bcf	txflags,RFREED,A
	bcf	INTCON3,INT2IF,A
	bsf	INTCON3,INT2IE,A
;	bcf	flags2,TEST1,A
	call	oscdown2
	bra	popret
;******************************************************************************
;Start of main program
; The main program code is placed here.

Main:
	 
	movlb	0
	bcf	OSCCON2,PRI_SD,A
	movlw	0xcf
	movwf	TRISC,A
	bcf	INTCON2,INTEDG1,A	;falling edge
	movlw	0x43	;internal oscillator, divide to 2MHz
	movwf	OSCCON,A
	bsf	OSCTUNE,INTSRC,A
	movlw	0x83	;16bit mode, presc. = 16, 31250Hz, ~ 2sec full cycle time
	movwf	T0CON,A
	movlw	0x0f	;TMR1 prescaler=1, ext.Osc=32768Hz
	movwf	T1CON,A
	bcf	LATB,TX,A
	bsf	TRISB,TX,A
	bcf	TRISB,SCK,A
	ifdef	debug
	bcf	TRISB,SDI,A	;for debug LED
	bsf	GREENLED
	bsf	REDLED
	endif	;debug
	lfsr	FSR0,0
	movlw	0
nulloop
	movwf	POSTINC0,A
	movwf	POSTINC0,A
	decfsz	0,F,A
	bra	nulloop
	movlw	0x0b	;A/D trigger mode
	movwf	CCP1CON,A
	movlw	0x89
	movwf	T3CON,A
	clrf	CCPR1H,A
	movlw	.50	;for 10KHz A/D rate at 2MHz Fosc
	movwf	CCPR1L,A
	movlw	0x25	;TMR2 prescaler=4, postsc.=5
	movwf	T2CON,A
	movlw	.250	;TMR2 interrupt 100Hz
	movwf	PR2,A
	clrf	flags,A
	bcf	ANSEL,ANS0,A	;INT0 input enable
	bcf	ANSEL,ANS2,A	;INT2 input enable
	bcf	ANSEL,ANS1,A	;INT1 input enable
	bcf	ANSELH,ANS9,A	;RC7 - RFPWR pin input enable
	movlw	0	;Vdd, Vss are references for A/D
	movwf	ADCON1,A
	movwf	ADCON0,A
	movlw	0xa0	;right justif. ,8TAD Acq. time, A/D clock= Fosc/2
	movwf	ADCON2,A
	bsf	TXSTA,BRGH,A
	movlw	0x00
	movwf	RCSTA,A
	movlw	0x08	;not inverted TX/RX, 16bit BRG
	movwf	BAUDCON,A
	movlw	high(.52) ;9600
	movwf	SPBRGH,A
	movlw	low(.52)
	movwf	SPBRG,A
	bcf	ANSELH,ANS11,A
	bcf	RCSTA,SPEN,A
	clrf	flags,A
	clrf	flags2,A
	clrf	flags3,A
	clrf	humflags,A
	clrf	txflags,A
	clrf	secctr,A
	clrf	minctr,A
	clrf	hrctr,A
	clrf	dayctr,A
	clrf	wormctrl,A
	clrf	wormctrh,A
	clrf	lastctrl,A
	clrf	lastctrh,A
	clrf	noise1ctrl,A
	clrf	noise1ctrh,A
	clrf	pulselenl,A
	clrf	pulselenh,A
	setf	wormlen,A
	clrf	recctr,A
	lfsr	FSR0,wormbuf
	ifdef	FastDebug
	movlw	.1	;1. timed measurement 1 minute after switch-on
	movwf	measctr,A
	movlw	.2	;measurements every 2 minutes for faster test
	else
	movlw	.10	;1. timed measurement 10 minutes after switch-on
	movwf	measctr,A
	movlw	.60	;measurements every 60 minutes
	endif
	movwf	meastc,A
	movlw	.30
	movwf	simtmr,A
	movlw	SIMULCOUNT	;10 simulations, then return to normal mode
	movwf	simctr,A
	clrf	WPUA,A
	bsf	WPUA,WPUA1,A
	bsf	WPUA,WPUA2,A
	clrf	WPUB,A
	bsf	INTCON,PEIE,A
	bcf	INTCON2,RABPU,A
	bcf	INTCON3,INT2IF,A
	bcf	INTCON3,INT2IE,A
	ifndef	__DEBUG
	bcf	INTCON,INT0IF,A
	bsf	INTCON,INT0IE,A
	endif	;__DEBUG
	bcf	INTCON,TMR0IF,A
	bsf	INTCON,TMR0IE,A
	bcf	PIR1,TMR1IF,A
	bsf	PIE1,TMR1IE,A
	bcf	PIR1,ADIF,A
	bsf	PIE1,ADIE,A
	bsf	PIE1,RCIE,A
	bsf	INTCON,GIE,A
	bcf	OSCCON,IRCF1,A
	bcf	OSCCON,IRCF0,A
	movlw	.3	;1. transmit 3 minutes after start
	movwf	txtimerl,A
	clrf	txtimerh,A
	movlw	5	;wait 5 secs to settle switch-on transients
	movwf	secctr,A
	ifdef	debug
	bcf	GREENLED
	endif	;debug
startloop
	btfss	flags3,STARTED,A  ;switch-on delay
	bra	startloop
	bcf	REDLED
	bsf	OSCCON,IRCF2,A
	bcf	flags,WORM,A
	bcf	flags2,WORMLOCK,A
	btfss	PORTA,1,A
	bsf	INTCON2,INTEDG1,A
	btfsc	PORTA,1,A	
	bcf	INTCON2,INTEDG1,A
	clrf	thmidx,A
	bsf	flags2,MEASPRE,A ;start with measurements
mainloop
	movlw	3
	cpfslt	wasting,A
	movwf	wasting,A
	bcf	INTCON,GIE,A
	btfsc	flags2,WORMLOCK,A
	rcall	t0test
	bsf	INTCON,GIE,A
	btfsc	flags,WORM,A
	rcall	wormeval
	btfsc	humflags,HUMID,A
	rcall	humtask
	btfsc	txflags,TXGO,A	;continue TX
	rcall	txnext
	btfsc	txflags,STARTTX,A ;start data TX
	rcall	txstart
	btfsc	flags,MEASST,A
	rcall	measstart
	btfsc	flags,MEASRDY,A
	rcall	thmtask
	btfsc	flags2,THMRDY,A
	rcall	thmtask
	btfsc	flags3,RSRCRDY,A
	rcall	rsrctask
;	btfsc	humflags,HUMSTART,A
	btfsc	flags2,MEASPRE,A
	rcall	measprep
	btfsc	humflags,HUMING,A
	rcall	humcheck
	btfsc	flags,SWRFON,A
	rcall	swrfpwron
	btfss	txflags,RFREED,A	;do not check if already on
	rcall	reedtest		;test if REED Relay on RF modul
	btfsc	humflags,HUMADRDY,A
	rcall	humadtask
	btfsc	flags,SECCHG,A
	rcall	sectask
	btfsc	flags3,RESTART,A
	rcall	restart
	bra	mainloop
restart
	bcf	flags3,RESTART,A
	bcf	flags3,WBUF2,A
	lfsr	FSR0,wormbuf
	movlw	low (TXTC)
	movwf	txtimerl,A
	movlw	high (TXTC)
	movwf	txtimerh,A
	movlw	.1
	movwf	measctr,A
	clrf	secctr,A
	clrf	minctr,A
	clrf	hrctr,A
	clrf	dayctr,A
	clrf	wormctrl,A	;reset collembola counter if reed act.
	clrf	wormctrh,A
	clrf	limctr,A
	clrf	recctr,A
	return
reedtest
	btfsc	txflags,RFCYCLE,A
	return
	bsf	TRISC,RFPWR,A
	btfss	PORTC,RFPWR,A	;RF modul pulled up?
	return			;No, nothing to do
	bsf	txflags,RFREED,A	;Yes, REED activated
;	clrf	recctr,A
swrfpwron
	btfsc	txflags,RFCYCLE,A	;If RF is already on, return
	return
	bsf	OSCCON,IRCF2,A	;hi clock rate
	bsf	txflags,RFCYCLE,A
	movlw	.18
	movwf	rftimeout,A
	bcf	PIE1,TMR2IE,A	;just to be shure
	bcf	INTCON,GIE,A
	bcf	flags,SWRFON,A	;acknoledge RF switch on
	bsf	LATC,RFPWR,A	;RFPWR control up
	bcf	TRISC,RFPWR,A	;RFPWR control pin output
	bsf	INTCON,GIE,A
;	bcf	flags2,TEST1,A
	bsf	RCSTA,SPEN,A	;configure serial port pins
	bsf	RCSTA,CREN,A
	bsf	PIE1,RCIE,A
	ifdef lockout
	bcf	INTCON3,INT2IE,A
	endif
	return
t0test
	btfsc	OSCCON,IRCF2,A
	return
	movf	TMR0L,W,A
	iorwf	TMR0H,W,A
	btfsc	_Z
	return
	movf	TMR0H,W,A
	addlw	2
	btfsc	_C
	return
	bcf	flags2,WORMLOCK,A
	bcf	REDLED
	return
sectask
	bcf	flags,SECCHG,A
	btfss	flags3,SIMTX,A
	return
	decfsz	simtmr,F,A
	return
	movlw	.30
	movwf	simtmr,A
	decfsz	simctr,F,A
	bra	simula
	bcf	flags3,SIMTX,A
	movlw	SIMULCOUNT
	movwf	simctr,A
	bsf	flags3,RESTART,A
	return
simula
	bsf	OSCCON,IRCF2,A
	bsf	flags,WORM,A
	clrf	integ2u,A
	clrf	integ2h,A
	clrf	limctr,A
	return
humcheck
	movlw	1
	cpfsgt	tickctru,A
	return
	btfsc	LATC,RNG,A
	bra	humst2
	bcf	INTCON,INT0IE,A
	bcf	INTCON,INT0IF,A
	clrf	humflags,A
	bsf	humflags,HUMID,A
	bsf	humflags,HUMOVF,A
	setf	resil,A
	setf	resih,A
	return
humst2
	bcf	LATC,RNG,A
	bcf	humflags,HUMRNG,A
humst3
	bsf	humflags,HUMINI,A
	clrf	tickctru,A
	bcf	INTCON,INT0IF,A
	bsf	INTCON,INT0IE,A
	return
measprep
	btfsc	flags2,WORMING,A	;Do not meas. while detection
	return
	btfsc	txflags,RFCYCLE,A	;Do not measure while RF on
	return
	bcf	flags2,MEASPRE,A
	tstfsz	thmidx,A
	bra	measstart
;start humid meas.
	bcf	humflags,HUMID,A  ;to be shure
	bsf	flags2,MEASING,A
	bsf	LATC,MPWR,A
	bcf	INTCON,INT0IE,A	;disable INT0 while A/D test
	bsf	OSCCON,IRCF2,A
	bsf	ANSEL,ANS0,A	;disable INT0 input
	bcf	ADCON2,ADFM,A
	bcf	humflags,HUMADRDY,A
	setf	resil,A	;min. A/D result init.
	clrf	resih,A	;max. A/D result init.
	clrf	huml,A
	clrf	humh,A
	movlw	.0
	movwf	integctr,A
	movlw	.2
	movwf	addelay,A
	movlw	1	;CH0, ADON
	movwf	ADCON0,A
	clrf	tickctru,A
	bsf	humflags,HUMING,A
	bcf	LATC,RNG,A
	bcf	humflags,HUMRNG,A
	return
measstart
	btfsc	flags2,WORMING,A
	return
	bsf	flags2,MEASING,A
	bsf	OSCCON,IRCF2,A
	bcf	flags,MEASST,A
	bsf	LATC,MPWR,A
	clrf	thml,A
	clrf	thmh,A
	movlw	.64
	movwf	integctr,A
	movlw	THMCHS + 1
	movwf	ADCON0,A
	movlw	.32
	movwf	addelay,A
	return
wtoomouch
	bcf	flags2,WORMLOCK,A
	bra	oscdown
wormeval
	bcf	flags,WORM,A
	bcf	GREENLED
	infsnz	wormctrl,F,A
	incf	wormctrh,F,A
	movlw	MAXPERHOUR
	cpfslt	limctr,A
	bra	wtoomouch
	incf	limctr,F,A
	movlw	MAXRECORDS
	btfsc	flags3,WBUF2,A
	movlw	MAXREC2
	cpfslt	recctr,A
	bra	wormok5	;no more RAM for records
	bcf	_C
	rrcf	integ2u,F,A
	rrcf	integ2h,F,A
	rrcf	integ2l,F,A
	bcf	_C
	rrcf	integ2u,F,A
	rrcf	integ2h,F,A
	rrcf	integ2l,F,A
	movf	integ2u,W,A
	bz	wormok2
	movlw	0xff
	bra	wormok3
wormok2
	movf	integ2h,W,A
wormok3
	movff	dayctr,POSTINC0	;0. byte of wormbuf record: day
	movff	hrctr,POSTINC0	; 1. byte: hour
	movff	minctr,POSTINC0	; 2. byte: minute
	movff	secctr,POSTINC0 ; 3. byte: second
	movff	batttx,POSTINC0 ; 4.
	movff	wormctrl,POSTINC0 ; 5.
	movff	wormctrh,POSTINC0 ; 6.
	movwf	POSTINC0,A	;7. byte coll. size  
	incf	recctr,F,A
wormok5
	btfsc	flags3,WBUF2,A
	return
	movlw	0xfe
	bcf	INTCON,TMR0IE,A
	movwf	TMR0H,A
	movlw	0x80
	movwf	TMR0L,A
	bcf	INTCON,TMR0IF,A
	bsf	INTCON,TMR0IE,A
	bra	swrfpwron
thmtask
	bcf	flags2,MEASING,A
	bcf	flags2,THMRDY,A
	lfsr	FSR2,thmbuf
	movf	thmidx,W,A
	movff	thml,PLUSW2
	incf	WREG,F,A
	movff	thmh,PLUSW2
	incf	WREG,F,A
	movwf	thmidx,A
	sublw	.16	;8 thm values max.
	bc	meastask
	movlw	2
	subwf	thmidx,F,A
meastask
	bcf	flags,MEASRDY,A
	bcf	INTCON3,INT2IF,A
	bsf	INTCON3,INT2IE,A
oscdown
	movf	PORTB,W,A
	btfsc	txflags,RFCYCLE,A
	return
	btfsc	flags2,WORMING,A
	return
oscdown2
	btfsc	flags2,MEASING,A
	return
	bcf	OSCCON,IRCF2,A
	return
humtask
	bcf	humflags,HUMID,A
	movlw	1
	movwf	humu,A
	btfsc	LATC,RNG,A
	bra	humt2	;already hi range

	movf	resih,W,A
	btfss	_Z
	bra	humt22	;big enough
	movlw	.10
	cpfslt	resil,A
	bra	humt22	;big enough
	bsf	LATC,RNG,A	;small, switch on 100nF
	bsf	humflags,HUMRNG,A
	bsf	humflags,HUMSTAB,A
	bra	humst3
humt2
	bsf	humu,1,A
humt22
	btfsc	humflags,HUMOVF,A
	bra	humoverf
	btfss	LATC,RNG,A
	bra	humt3
	nop
	bra	humt3
humoverf
	setf	huml,A
	setf	humh,A
humt3
	movff	resil,humltx
	movff	resih,humhtx
	movff	humu,humutx
	bsf	LATC,RNG,A
	bsf	flags,MEASST,A
;	bcf	LATC,MPWR,A
	clrf	humflags,A
;	bra	humrstx
	return
humadtask
	bcf	humflags,HUMADRDY,A
	movlw	0xef
	cpfslt	resil,A
	bra	humhir
	movf	resil,W,A
	subwf	resih,W,A	;W=A/D max - A/D min.
	movwf	resil,A
	addlw	4
	bc	humhir
	clrf	resih,A
	clrf	humu,A
	bra	humt3
humhir
	bra	humst2
humrstx
	bsf	RCSTA,SPEN,A
	nop
	bsf	TXSTA,TXEN,A
	nop
	movff	humltx,TXREG
	btfss	TXSTA,TRMT,A
	bra	$-2
	movff	humhtx,TXREG
	btfss	TXSTA,TRMT,A
	bra	$-2
	movff	humutx,TXREG
	btfss	TXSTA,TRMT,A
	bra	$-2
	movff	testvall,TXREG
	btfss	TXSTA,TRMT,A
	bra	$-2
	movff	testvalh,TXREG
	btfss	TXSTA,TRMT,A
	bra	$-2
	bcf	TXSTA,TXEN,A
	return
rsrctask
	bcf	flags3,RSRCRDY,A
	movlw	0x21	;RF ack. to locale sw-on
	cpfseq	rscmd,A
	bra	tfacky
	movlw	0x5a
	subwf	lastrx,W,A
	bz	bug
	movlw	4
	movwf	txcode,A
	btfsc	txflags,RFREED,A
	bra	reenq
	bsf	txflags,STARTTX,A
	return
bug
	clrf	lastrx,A
	return
	movlw	.2
	movwf	rftimeout,A
	bcf	TXSTA,TXEN,A
	bcf	RCSTA,CREN,A
	bcf	RCSTA,SPEN,A
	bcf	LATC,RFPWR,A
	bcf	LATB,TX,A
	bcf	flags2,TEST1,A
	return	
reenq
	clrf	recctr,A
	bcf	flags3,WBUF2,A
	lfsr	FSR0,wormbuf
	movlw	.30
	movwf	simtmr,A
	movlw	SIMULCOUNT
	movwf	simctr,A
	bcf	flags2,TEST1,A
	bsf	flags3,SIMTX,A
	bsf	txflags,TXGO,A	;go on with next 32 bytes block
	clrf	limctr,A
	return
tfacky
	movlw	0x59	;normal RF TX done
	cpfseq	rscmd,A
	bra	tfreedon
	movwf	lastrx,A
;	btfsc	txflags,RFREED,A  ;if after REED test tx, it's done
;	bra	rftxdone
	btfsc	flags,RFDONE,A
	bra	rftxdone
	bsf	txflags,TXGO,A	;go on with next 32 bytes block
	movlw	.15
	movwf	rftimeout,A
	return
rftxdone
	movlw	.4
	movwf	rftimeout,A
	bcf	flags,RFDONE,A
	bcf	TXSTA,TXEN,A
	bcf	RCSTA,CREN,A
	bcf	RCSTA,SPEN,A
	bcf	LATC,RFPWR,A
	bcf	LATB,TX,A
	clrf	TMR2,A		;init tx-down timer
	bcf	PIR1,TMR2IF,A
	bsf	PIE1,TMR2IE,A
	movlw	.100
	movwf	tmr2ctr,A
	bcf	txflags,RFREED,A
	bcf	flags2,TEST1,A
	lfsr	FSR0,wormbuf
	clrf	recctr,A
	bcf	flags3,WBUF2,A
	ifdef	lockout
	bcf	INTCON3,INT2IF,A
	bsf	INTCON3,INT2IE,A
	endif
	return
moveback
	btfss	flags3,WBUF2,A
	return
	lfsr	FSR0,wormbuf
	movf	recctr,W,A
	btfsc	_Z
	bra	rftxdone2
	movwf	temp,A
	lfsr	FSR2,wormbuf2
bmovloope
	movlw	RECLEN
bmovloopi
	movff	POSTINC2,POSTINC0
	decfsz	WREG,F,A
	bra	bmovloopi
	decfsz	temp,F,A
	bra	bmovloope
	bsf	flags,SWRFON,A
rftxdone2
	bcf	flags3,WBUF2,A
	return
tfreedon
	movlw	0x31	;reed ack. each 6 sec.
	cpfseq	rscmd,A
	bra	tfreedack
	movwf	lastrx,A
	movlw	5
	movwf	txcode,A
	bsf	txflags,RFREED,A
	bsf	flags3,SIMTX,A
;	btfss	flags2,TEST1,A
	bsf	txflags,TXGO,A	;send 32 bytes test block
	return
tfreedack
	movlw	0x79	;test RF TX done
	cpfseq	rscmd,A
	bra	tf5ah
	movwf	lastrx,A
	bsf	txflags,TXGO,A	;send 32 bytes test block
;	bcf	flags3,SIMTX,A
	return
tf5ah
	movlw	0x5a	;normal data ack.
	cpfseq	rscmd,A
	bra	tfother
	movwf	lastrx,A
	movlw	.15
	movwf	rftimeout,A
	return	;todo: ??
tf7ah
	movlw	0x7a	;test data ack.
	cpfseq	rscmd,A
	bra	tfother
	movwf	lastrx,A
	movlw	.15
	movwf	rftimeout,A
	return
tfother
	return
delay
	movlw	.250
	movwf	delayctr,A
delloop
	decfsz	delayctr,F,A
	bra	delloop
	return	

longdelay
	movwf	delayctr,A
	clrf	WREG,A
delayloop
	decfsz	WREG,F,A
	bra	delayloop
	decfsz	delayctr,F,A
	bra	delayloop
	return
eeread		;addr. in W
	MOVWF	EEADR,A ; Data Memory Address to read
eereada
	BCF	EECON1,EEPGD,A ; Point to DATA memory
	BCF	EECON1,CFGS,A ; Access program FLASH or Data EEPROM memory
	BSF	EECON1,RD,A ; EEPROM Read
	MOVF	EEDATA,W,A
	return
eewrite		;addr. in W, data should heve been loaded into EEDATA
	btfsc	EECON1,WR,A	;wait previous write to complete, if any
	bra	eewrite
	bcf	PIR2,EEIF,A	; to be sure
	MOVWF	EEADR,A	; Data Memory Address to Write
	movlw	4	;WREN
	movwf	EECON1,A
	BCF	INTCON,GIE,A ; Disable interrupts
	MOVLW	55h
	MOVWF	EECON2,A ; Write 55h
	MOVLW	0AAh
	MOVWF	EECON2,A ; Write AAh
	BSF	EECON1,WR,A ; Set WR bit to begin write
	BSF	INTCON,GIE,A ; Enable interrupts
	BTFSC	EECON1,WR,A ; Wait for write to complete
	BRA	$-2
	return
fillframe
	lfsr	FSR1,txbuf
	movlw	0x55	;Sync 0.
	movwf	POSTINC1,A
	movlw	0xaa	;sync 1.
	movwf	POSTINC1,A
	rcall	getID
	movlw	high EdaphoVersion	;Ver. 2. 4.
	movwf	POSTINC1,A
	movlw	low EdaphoVersion	;ver. .2 5.
	movwf	POSTINC1,A
	return
txnext
	bcf	txflags,TXGO,A
	btfss	txflags,RFREED,A
	bra	txprep
testtx		;reed relay activated
	clrf	wormctrl,A	;reset collembola counter if reed act.
	clrf	wormctrh,A
	bsf	flags,RFDONE,A
	btfsc	flags2,TEST1,A
	bra	testcounter
	rcall	fillframe
	movff	dayctr,POSTINC1	; 6. byte
	movff	hrctr,POSTINC1	;7.
	movff	minctr,POSTINC1	;8.
	movff	secctr,POSTINC1 ;9. byte
	movf	batttx,W,A
	movwf	POSTINC1,A	;10. batt
	movff	wormctrl,POSTINC1 ; 11. byte: coll. ctr.
	movff	wormctrh,POSTINC1 ; 12.
	movlw	0x00
	movwf	POSTINC1,A	; 13. byte: coll. size
	rcall	geteda
	bsf	flags2,TEST1,A
	bcf	txflags,RFREED,A
testcounter
	lfsr	FSR1,txbuf + .30
	movff	rfctr,POSTDEC1
testctrlp
	movlw	0x39		;BCD counter in test sting
	cpfseq	INDF1,A
	bra	testctr2
	movlw	0x30
	movwf	POSTDEC1,A
	bra	testctrlp
testctr2
	incf	INDF1,F,A
	bra	txprep8		;start TX

txstart
	bcf	txflags,STARTTX,A
	movff	recctr,txrecctr
	clrf	recctr,A
	lfsr	FSR0,wormbuf2
	bsf	flags3,WBUF2,A
	lfsr	FSR2,wormbuf	;------ FSR2 points to wormbuf --------
txprep
	rcall	fillframe	;FSR1 points to txbuf 6. byte
	movf	txrecctr,W,A
	btfss	_Z
	bra	txprep2
	incf	txrecctr,F,A	;txrecctr=1
	movff	dayctr,POSTINC1	; 6. byte
	movff	hrctr,POSTINC1	;7.
	movff	minctr,POSTINC1	;8.
	movff	secctr,POSTINC1 ;9. byte
	movf	batttx,W,A
	movwf	POSTINC1,A	;10. batt
	movff	wormctrl,POSTINC1 ; 11. byte: coll. ctr.
	movff	wormctrh,POSTINC1 ; 12.
	movlw	0x00
	movwf	INDF1,A	; 13. byte: coll. size
	bra	txprep5	
txprep2
	movlw	5
	movwf	temp,A
txprep3
	movf	POSTINC2,W,A	;get worm record
	movwf	POSTINC1,A	;6 to 9 timestamp, 10 batt.
	decfsz	temp,F,A
	bra	txprep3
	movf	POSTINC2,W,A	;11. worm count low
	movwf	lastctrl,A	;save for interim reports
	movwf	POSTINC1,A	;
	movf	POSTINC2,W,A	;12. worm count high
	movwf	lastctrh,A
	movwf	POSTINC1,A
	movff	POSTINC2,INDF1 ;13. size
txprep5
	movf	POSTINC1,W,A
	bnz	txprep6
	lfsr	FSR2,thmbuf
	movlw	.16
	movwf	temp,A
txprep55
	movff	INDF2,POSTINC1
	clrf	POSTINC2,A
	decfsz	temp,F,A
	bra	txprep55
	clrf	thmidx,A
	bra	txprep77
txprep6
	movf	thmltx,W,A	;14. thm
	movwf	POSTINC1,A
	movf	thmhtx,W,A	;15.
	movwf	POSTINC1,A
	movf	humltx,W,A	;16. hum
	movwf	POSTINC1,A
	movf	humhtx,W,A	;17.
	movwf	POSTINC1,A
	movf	humutx,W,A	;18.
	movwf	POSTINC1,A
	movf	noise1ctrl,W,A	;19. noise1 low
	movwf	POSTINC1,A
	movf	noise1ctrh,W,A	;20. noise1 high
	movwf	POSTINC1,A
	movf	testvall,W,A ;uxltx	;21. Ux low
	movwf	POSTINC1,A
	movf	testvalh,W,A ;uxhtx	;22. Ux high
	movwf	POSTINC1,A
	movlw	.04
	movwf	temp,A		;23-26.
	movlw	0
txprep7
	movwf	POSTINC1,A
	decfsz	temp,F,A
	bra	txprep7
	movff	opdcltx,POSTINC1 ;27.
	movff	opdchtx,POSTINC1 ;28.
	movff	txrecctr,POSTINC1  ;29.
txprep77
	movff	rfctr,POSTINC1 ;30.
	decf	txrecctr,F,A
	movlw	0x17		;31.
	movwf	INDF1,A
	bnz	txprep8
	movf	txcode,W,A		;31.
	movwf	INDF1,A
	bsf	flags,RFDONE,A
txprep8
	lfsr	FSR1,txbuf
	movlw	.32
	movwf	txctr,A
	incf	rfctr,F,A
	bsf	RCSTA,SPEN,A
	bsf	TXSTA,TXEN,A
	bsf	PIE1,TXIE,A
	btfss	flags,RFDONE,A
	return
	lfsr	FSR0,wormbuf
	clrf	recctr,A
	return
	bra	moveback
edaphostr dw	" EDAPHOLOG #0000",0x0523
geteda
	movlw	low edaphostr
	movwf	TBLPTRL,A
	movlw	high edaphostr
	movwf	TBLPTRH,A
	clrf	TBLPTRU,A
	movlw	.18
	movwf	temp,A
getedalp
	tblrd*+
	movff	TABLAT,POSTINC1
	decfsz	temp
	bra	getedalp
	return
getID
	movlw	low pUnicID
	movwf	TBLPTRL,A
	movlw	high pUnicID
	movwf	TBLPTRH,A
	clrf	TBLPTRU,A
	tblrd*+
	movff	TABLAT,POSTINC1
	tblrd*+
	movff	TABLAT,POSTINC1
	return
	
;******************************************************************************
;End of program

		END
