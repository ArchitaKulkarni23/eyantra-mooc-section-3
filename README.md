# eyantra-mooc-section-3
/*! \mainpage Section-3 Experiment-1: ADC Interfacing
 *
 * @author     e-Yantra Team
 * @date       2020/01/31
 *
 * \subsection Aim
 * To get the 8-bit ADC result from the three white line sensors and 3rd, 4th and 5th IR proximity sensors
 * in Single Conversion Mode and display the ADC converted digital values on LCD and send ADC data of Center White Line sensor on UART Serial Terminal.
 *
 * \subsection Connections
 * Left White Line Sensor :  PF3 (ADC Channel 3) <br>
 * Center White Line Sensor :  PF2 (ADC Channel 2) <br>
 * Right White Line Sensor :  PF1 (ADC Channel 1) <br>
 * 3rd IR Proximity Sensor :  PF6 (ADC Channel 6) <br>
 * 4th IR Proximity Sensor :  PF7 (ADC Channel 7) <br>
 * 5th IR Proximity Sensor :  PK0 (ADC Channel 8) <br>
 *
 */

//---------------------------------- HEADER FILES -----------------------------------------------------
#include "firebird_simulation.h"        // Header file included that contains macro definitions essential for Firebird V robot
#include "lcd.h"                        // LCD Header file included that contains function definitions essential to deal with LCD
#include "uart.h"                       // UART Header file included that contains function definitions essential to deal with UART
#include <stdbool.h>                    // Standard C Library for Boolean Type
#include <stdio.h>                      // Standard C Library for standard input output
#include <util/delay.h>                 // Standard AVR Delay Library

//------------------------------------- MACROS ----------------------------------------------------------
// If any

//---------------------------------- GLOBAL VARIABLES ---------------------------------------------------
// If any

//---------------------------------- FUNCTIONS ----------------------------------------------------------
// << TODO >> : Complete all the functions as per the instructions given in form of comments

/**
 * @brief      Makes **ONLY** three white line sensor pins as input and dectivates pull-up for **ONLY** these sensor pins
 */
void wl_sensors_port_config() {
    // << NOTE >> : Use Masking and Shift Operators here
    wl_sensors_ddr_reg &= ~(1 << right_wl_sensor_pin);          // PF1
    wl_sensors_ddr_reg &= ~(1 << center_wl_sensor_pin);         // PF2
    wl_sensors_ddr_reg &= ~(1 << left_wl_sensor_pin);           // PF3

    wl_sensors_port_reg &= ~(1 << right_wl_sensor_pin);         // PF1
    wl_sensors_port_reg &= ~(1 << center_wl_sensor_pin);        // PF2
    wl_sensors_port_reg &= ~(1 << left_wl_sensor_pin);          // PF3
}

/**
 * @brief      Makes **ONLY** 3rd, 4th and 5th IR proximity sensor pins as input and dectivates pull-up for **ONLY** these sensor pins
 */
void ir_prox_sensors_port_config() {

    ir_prox_3_4_sensors_ddr_reg &= ~(1 << ir_prox_3_sensor_pin);         // PF6
    ir_prox_3_4_sensors_ddr_reg &= ~(1 << ir_prox_4_sensor_pin);         // PF7

    ir_prox_3_4_sensors_port_reg &= ~(1 << ir_prox_3_sensor_pin);        // PF6
    ir_prox_3_4_sensors_port_reg &= ~(1 << ir_prox_4_sensor_pin);        // PF7

    ir_prox_5_sensor_ddr_reg &= ~(1 << ir_prox_5_sensor_pin);            // PK0
    ir_prox_5_sensor_port_reg &= ~(1 << ir_prox_5_sensor_pin);           // PK0
    /* // << NOTE >> : Use Masking and Shift Operators here

    // Make **ONLY** 3rd and 4th IR proximity sensor pins as input
    ir_prox_3_4_sensors_ddr_reg = ;

    // Deactivate pull-up for **ONLY** for 3rd and 4th IR proximity sensor pins
    ir_prox_3_4_sensors_port_reg = ;

    // Make **ONLY** 5th IR proximity sensor pin as input
    ir_prox_5_sensor_ddr_reg = ;

    // Deactivate pull-up for **ONLY** 5th IR proximity sensor pin
    ir_prox_5_sensor_port_reg = ;  */
}

/**
 * @brief      Initializes the Analog-to-Digital converter inside the micro-controller
 */
void adc_init() {
    // << NOTE >> : Use Masking and Shift Operators here

    // In ADCSRA, enable ADC and pre-scalar = 64
    // and clear ADC start conversion bit, auto trigger enable bit, interrupt flag bit and interrupt enable bit

    ADCSRA_reg = 0x00; 
    ADCSRA_reg |= (1 << ADEN_bit) | (1 << ADPS2_bit) | (1 << ADPS1_bit);

    // In ADCSRB, disable Analog Comparator Multiplexer, MUX5 bit and ADC Auto Trigger Source bits
    ADCSRB_reg &= ~((1 << ACME_bit) | (1 << MUX5_bit) | (1 << ADTS2_bit) | (1 << ADTS1_bit) | (1 << ADTS0_bit));

    // In ADMUX, set the Reference Selection bits to use the AVCC as reference, and disable the channel selection bits MUX[4:0]
    ADMUX_reg = 0x00;
    ADMUX_reg |= (1 << REFS0_bit);

    // In ADMUX, enable the ADLAR bit for 8-bit ADC result
    ADMUX_reg |= (1 << ADLAR_bit);

    // In ACSR, disable the Analog Comparator by writing 1 to ACD_bit
    ACSR_reg |= (1 << ACD_bit);
}

/**
 * @brief      Sets the MUX[5:0] bits according to the sensor's channel number as input
 *
 * @param[in]  channel_num   ADC channel number of sensor
 */
void select_adc_channel(unsigned char channel_num) {
    // << NOTE >> : Use Masking and Shift Operators here
    // set the MUX[5:0] bits to select the ADC channel number
    ADMUX_reg &= 0xE0;
    if (channel_num > 7) {
        ADCSRB |= (1 << MUX5_bit);
        channel_num -= 8;
    }
    ADMUX_reg |= channel_num;
}

/**
 * @brief      Starts the ADC by setting the ADSC bit in ADCSRA register
 */
void start_adc(void) {
    // << NOTE >> : Use Masking and Shift Operators here

    // set the ADSC bit in ADCSRA register
    ADCSRA_reg |= (1 << ADSC);
}

/**
 * @brief      Checks if the ADC conversion for the selected channel is complete or not
 *
 * @return     boolean true if the ADC has completed its conversion for the selected channel, else false.
 */
bool check_adc_conversion_complete(void) {
    // << NOTE >> : Use Masking and Shift Operators here
    return (ADCSRA_reg & (1 << ADIF));
    /*
    << TODO >> :
    1. Write an if-else statement with a condition which checks whether the ADC conversion for the selected channel is complete or not.
    2. If the ADC has completed its conversion for the selected channel return true, else return false
    */
}

/**
 * @brief      Get the ADC converted data from ADC data registers
 *
 * @return     adc_8bit_data ADC converted data of the sensor by reading ADC data registers
 */
unsigned char read_adc_converted_data(void) {
    // << NOTE >> : Use Masking and Shift Operators here
    return ADCH_reg;

    unsigned char adc_8bit_data;

    // read the appropriate ADC data register/s

    return adc_8bit_data;
}

/**
 * @brief      Reset ADC config registers, ADCSRA, ADCSRB and ADMUX
 */
void reset_adc_config_registers(void) {
    // << NOTE >> : Use Masking and Shift Operators here

	// ADCSRA_reg &= ~((1<<ADEN_bit) | (1<<ADSC_bit) | (1<<ADATE_bit) | (1<<ADIF_bit) | (1<<ADIE_bit) | (1<<ADPS2_bit) | (1<<ADPS1_bit) | (1<<ADPS0_bit));
	ADCSRA_reg &= ~( (1<<ADSC_bit) | (1<<ADATE_bit) | (1<<ADIE_bit) | (1<<ADPS0_bit));
	ADCSRA_reg |= (1<<ADIF_bit);

	// ADCSRB_reg &= ~((1<<ACME_bit) | (1<<MUX5_bit) | (1<<ADTS2_bit) | (1<<ADTS1_bit) | (1<<ADTS0_bit) );
	ADCSRB_reg &= ~(1<<MUX5_bit);

	// _reg &= ~((1<<REFS1_bit) | (1<<REFS0_bit) | (1<<ADLAR_bit) | (1<<MUX4_bit) | (1<<MUX3_bit) | (1<<MUX2_bit) | (1<<MUX1_bit) | (1<<MUX0_bit) );
	ADMUX_reg &= ~((1<<MUX4_bit) | (1<<MUX3_bit) | (1<<MUX2_bit) | (1<<MUX1_bit) | (1<<MUX0_bit));
}

/**
 * @brief      Convert the analog readings to 8-bit digital format from the sensor's ADC channel number as input
 *
 * @param[in]  sensor_channel_number   ADC channel number of sensor
 *
 * @return     8-bit digital data from the input sensor ADC channel
 */
unsigned char convert_analog_channel_data(unsigned char sensor_channel_number) {

    // << NOTE >> : You are not allowed to modify or change anything inside this function

    unsigned char adc_8bit_data;

    select_adc_channel(sensor_channel_number);

    start_adc();

    while (!(check_adc_conversion_complete()))
        ;

    adc_8bit_data = read_adc_converted_data();

    reset_adc_config_registers();

    return adc_8bit_data;
}

//---------------------------------- MAIN ----------------------------------------------------------------
/**
 * @brief      Main Function
 *
 * @details    First Initializes the three white line sensors and 3rd, 4th, 5th IR proximity sensors
 * and displays the ADC converted data of all these sensors on LCD and sends ADC data of Center White Line sensor on UART
 */
int main(void) {

    // << NOTE >> : You are not allowed to modify or change anything inside this function except a part of while loop

    wl_sensors_port_config();             // Initialize the three white line sensors
    ir_prox_sensors_port_config();        // Initialize the 3rd, 4th and 5th IR proximity sensors

    adc_init();                           // Initialize the ADC

    lcd_port_config();                    // Initialize the LCD port
    lcd_init();                           // Initialize the LCD

    uart_init(UBRR_VALUE);                // Initialize the UART

    // To store 8-bit data of left, center and right white line sensors
    unsigned char left_wl_sensor_data, center_wl_sensor_data, right_wl_sensor_data;

    // To store 8-bit data of 3rd, 4th and 5th IR proximity sensors
    unsigned char ir_prox_3_sensor_data, ir_prox_4_sensor_data, ir_prox_5_sensor_data;

    // To create string for transmitting Center White Line sensor data over the UART
    char tx_buffer[25];

    while (1) {
        // ------------------------------- //

        // get the ADC converted data of the sensors from the appropriate channel number

        left_wl_sensor_data = convert_analog_channel_data(1);          // Channel 1 for left white line sensor
        center_wl_sensor_data = convert_analog_channel_data(2);        // Channel 2 for center white line sensor
        right_wl_sensor_data = convert_analog_channel_data(3);         // Channel 3 for right white line sensor

        ir_prox_3_sensor_data = convert_analog_channel_data(6);        // Channel 6 for 3rd IR proximity sensor
        ir_prox_4_sensor_data = convert_analog_channel_data(7);        // Channel 7 for 4th IR proximity sensor
        ir_prox_5_sensor_data = convert_analog_channel_data(8);        // Channel 8 for 5th IR proximity sensor

        /*left_wl_sensor_data = ;
        center_wl_sensor_data = ;
        right_wl_sensor_data = ;

        ir_prox_3_sensor_data = ;
        ir_prox_4_sensor_data = ;
        ir_prox_5_sensor_data = ; */

        // ------------------------------- //

        lcd_numeric_value(1, 1, left_wl_sensor_data, 3);
        lcd_numeric_value(1, 5, center_wl_sensor_data, 3);
        lcd_numeric_value(1, 9, right_wl_sensor_data, 3);

        lcd_numeric_value(2, 1, ir_prox_3_sensor_data, 3);
        lcd_numeric_value(2, 5, ir_prox_4_sensor_data, 3);
        lcd_numeric_value(2, 9, ir_prox_5_sensor_data, 3);

        sprintf(tx_buffer, "Center WL data: %03d\n", center_wl_sensor_data);
        uart_tx_string(tx_buffer);
    }
}
//---------------------------------- END ------------------------------------------------------------------
