## PRIORIDAD DE VISUALIZACIÓN SPRITE/FONDO

Los sprites pueden aparecer delante o detrás de la escena de fondo. Esto se controla mediante el bit 12 del código de carácter de fondo de 16 bits. Si este bit se establece en 0, todos los sprites aparecen sobre el fondo. Si este bit se establece en 1, los colores de fondo #1-15 aparecen sobre los sprites. El color de fondo #0 siempre aparece debajo de los sprites.

