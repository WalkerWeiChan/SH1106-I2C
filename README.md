# SH1106-I2C
SH1106 OLED display driver for STM32 using I2C HAL


The driver was built focusing on a  simple and intuitive interface, easy to use and flexible, similar to the existing GFX libraries, but not too fancy.




Hi Paulo R. Santos 

Thank you for sharing your source code. I found some aspects worth optimizing during the usage.
1. In the function "send_page", you can simply add a 0x40 at the beginning, followed by 128 bytes of data. I suggest changing it to this:
HAL_I2C_Mem_Write(&hi2c2, SH1106_ADDRESS << 1, 0x40, 1, (uint8_t*)page_data, 128, 20);

2.In the function "draw_vertical_line_base", there is something wrong of it. In the switch(self.pixel_mode), case SH1106_PSET, it need to send 0xFF instead of 0x00, case SH1106_PRES, it need to send 0x00 instead of 0xFF. Like below:
switch (self.pixel_mode) {
    case SH1106_PSET:
        for (; h > 7; h -= 8) {
            *ptr = 0xFF;
            ptr += SCR_W;
        }
        break;
    case SH1106_PRES:
        for (; h > 7; h -= 8) {
            *ptr = 0x00;
            ptr += SCR_W;
        }
        break;


3. In the function "print_char", if use a font which higher than 8 or wider than 8, it will get a worng display. I change to like below.

static uint8_t print_char(uint8_t x, uint8_t y, char character, const font_t* fnt) {
    const uint8_t* char_bmp;
    uint8_t x_limit;
    uint8_t y_limit;
    uint8_t temp,i,j;

    if (character < fnt->min_char || character > fnt->max_char) {
        character = fnt->max_char;
    }

    switch (fnt->scan) {
    case FONT_V:
        char_bmp = &fnt->characters[((fnt->height + 7)>>3) * (character - fnt->min_char) \
                                    * fnt->width];
        for (x_limit = x + fnt->width; x < x_limit; x++) 
        {
            temp = y;
            i = 0;
            for(j = fnt->height; j > 0 ; j--)
            {
                if (*char_bmp & (1<<i))
                {
                    draw_pixel(x, temp);
                }
                temp++;
                if(i<7)
                {
                    i++;
                }
                else if(j>1)
                {
                    i = 0;
                    char_bmp++;
                }
            }
            char_bmp++;
        }
        break;
    case FONT_H:
        char_bmp = &fnt->characters[((fnt->width + 7)>>3) * (character - fnt->min_char) \
                                    * fnt->height];

        for (y_limit = y + fnt->height; y < y_limit; y++) 
        {
            temp = x;
            i = 0;
            for(j = fnt->width; j > 0 ; j--)
            {
                if (*char_bmp & (1<<i))
                {
                    draw_pixel(temp, y);
                }
                temp++;
                if(i<7)
                {
                    i++;
                }
                else if(j>1)
                {
                    i = 0;
                    char_bmp++;
                }
            }
            char_bmp++;
        }
        break;
    default:
        break;
    }

    return fnt->width + 1;
}
