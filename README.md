ESP8266 SPI Driver
========

Some general information about the SPI hardware and my investigations into the control registers are on http://d.av.id.au/blog/ for those who want a deeper insight into how the code works.

HSPI Hardware Pins
========

|Pin Name| GPIO # | HSPI Function |
|--------|--------|---------------|
|MTDI	 | GPIO12 | MISO (DIN)    |
|MTCK	 | GPIO13 | MOSI (DOUT)   |
|MTMS	 | GPIO14 | CLOCK         |
|MTDO	 | GPIO15 | CS / SS       |

Main SPI Transaction Function
========
Perform an SPI transaction. 

spi_transaction(spi_no, cmd_bits, cmd_data, addr_bits, addr_data, dout_bits, dout_data, din_bits, dummy_bits)

spi_no: SPI or HSPI

cmd_bits: length of command. Set to 0 to disable command component.

cmd_data: Command data to send.

addr_bits: length of address. Set to 0 to disable address component.

addr_data: Address data to send.

dout_bits: Length of data payload to send. Set to 0 to disable data out component.

dout_data: Data payload to send.

din_bits: Length of data payload to receive. Set to 0 if not receiving any data.

dummy_bits: Number of dummy bits to insert.

Data Structure of SPI Transaction

[COMMAND]+[ADDRESS]+[DataOUT]+[DUMMYBITS]+[DataIN]

If no data is set to be received, then the Dummy bits are inserted before DataOUT.

[COMMAND]+[ADDRESS]+[DUMMYBITS]+[DataOUT]

Example:


Writing data to EEPROM

Command = 0b101 (3 bit write command)

Address = 0b111110011 or 0x1F3 (9 bit data address)

Data    = 0b11001100 or 0xCC (8 bits of data) 

SPI Transaction Packet = 0b10111111001111001100 or 0xBF3CC

spi_transaction(HSPI, 3, 0b101, 9, 0x1F3, 8, 0xCC, 0, 0);


Reading data from EEPROM

Command = 0b110 (3 bit read command)

Address = 0b111110011 or 0x1F3 (9 bit data address)

(8 bits of data to read)

SPI Transaction Packet = 0b11011111001100000000 or 0xDF300 //last 8 zeros are to clock 8 bits from the EEPROM

data_read = (uint8) spi_transaction(HSPI, 3, 0b101, 9, 0x1F3, 0, 0, 8, 0);


Basic Transmit Functions
========

Send 'n' bits of data:
spi_txd(spi_no, bits, data)

Send 8 bits of data (1 BYTE):
spi_tx8(spi_no, data)

Send 16 bits of data (1 WORD):
spi_tx16(spi_no, data)

Send 32 bits of data (1 DWORD):
spi_tx32(spi_no, data) 

Example:

uint8 byte = 0xEF;

uint16 word = 0xBEEF;

uint32 dword = 0xDEADBEEF;

uint16 9bit = 0b101010101;

spi_tx8(HSPI, byte);

spi_tx16(HSPI, word);

spi_tx32(HSPI, dword);

spi_txd(HSPI, 9, 9bit);


Basic Receive Functions
========

Receive 'n' bits of data:
spi_rxd(spi_no, bits)

Receive 8 bits of data (1 BYTE): 
spi_rx8(spi_no)

Receive 16 bits of data (1 WORD): 
spi_rx16(spi_no)

Receive 32 bits of data (1 DWORD): 
spi_rx32(spi_no) 

Example:

uint8 byte;

uint16 word;

uint32 dword;

uint16 9bit;

byte = (uint8) spi_rx8(HSPI); //returned value is uint32. Cast to uint8

word = (uint16) spi_rx16(HSPI); //returned value is uint32. Cast to uint16

dword = spi_rx32(HSPI); //No type casting needed

9bit = (uint16) spi_rxd(HSPI, 9); //returned value is uint32. Cast to uint16



