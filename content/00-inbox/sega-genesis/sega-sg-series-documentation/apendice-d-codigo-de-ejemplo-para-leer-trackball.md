# Apéndice D - Código de Ejemplo para Leer el Trackball

```assembly
 1:
 2: ;~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~;
 3: ;  LECTURA DE INTERRUPTORES DEL TRACKBALL                          ;
 4: ;  RET P1_TRX ~ CONDICIÓN JUGADOR 1 (-7FH ~ 7FH)                   ;
 5: ;      P1_TRY ~ CONDICIÓN JUGADOR 1 (-7FH ~ 7FH)                   ;
 6: ;      P2_TRX ~ CONDICIÓN JUGADOR 2 (-7FH ~ 7FH)                   ;
 7: ;      P2_TRY ~ CONDICIÓN JUGADOR 2 (-7FH ~ 7FH)                   ;
 8: ;~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~;
 9: 01D0' T_BALL::
 10: 01D0' TRACK_X1:
 11: 01D0' 3E 0D        LD A,00001101B
 12: 01D2' D3 3F        OUT (PSWC),A
 13: 01D4' 06 19        LD B,25
 14: 01D6' 10 FE        DJNZ $
 15: 01D8' DB DC        IN A,(P1_SWPT)
 16:
 19: 01DA' E6 0F        AND 0FH
 20:                    REPT 4
 21:                    RRCA
 22:                    ENDM
 23: 01DC' 0F         * RRCA
 24: 01DD' 0F         * RRCA
 25: 01DE' 0F         * RRCA
 26: 01DF' 0F         * RRCA
 27: 01E0' 57           LD D,A
 28: 01E1' 3E 2D        LD A,00101101B
 29: 01E3' D3 3F        OUT (PSWC),A
 30: 01E5' 06 0D        LD B,13
 31: 01E7' 10 FE        DJNZ $
 32: 01E9' DB DC        IN A,(P1_SWPT)
 33: 01EB' E6 0F        AND 0FH
 34: 01ED' B2           OR D
 35: 01EE' ED 44        NEG
 36: 01F0' 32 C018      LD (P1_TRX),A
 37: 01F3' TRACK_Y1:
 38: 01F3' 3E 0D        LD A,00001101B
 39: 01F5' D3 3F        OUT (PSWC),A
 40: 01F7' 06 0D        LD B,13
 41: 01F9' 10 FE        DJNZ $
 42: 01FB' DB DC        IN A,(P1_SWPT)
 43: 01FD' E6 0F        AND 0FH
 44:                    REPT 4
 45:                    RRCA
 46:                    ENDM
 47: 01FF' 0F         * RRCA
 48: 0200' 0F         * RRCA
 49: 0201' 0F         * RRCA
 50: 0202' 0F         * RRCA
 51: 0203' 57           LD D,A
 52: 0204' 3E 2D        LD A,00101101B
 53: 0206' D3 3F        OUT (PSWC),A
 54: 0208' 06 0D        LD B,13
 55: 020A' 10 FE        DJNZ $
 56: 020C' DB DC        IN A,(P1_SWPT)
 57: 020E' E6 0F        AND 0FH
 58: 0210' B2           OR D
 59: 0211' ED 44        NEG
 60: 0213' 32 C019      LD (P1_TRY),A
 61: 0216' TRACK_X2:
 62: 0216' 3E 07        LD A,00000111B
 63: 0218' D3 3F        OUT (PSWC),A
 64: 021A' 06 19        LD B,25
 65: 021C' 10 FE        DJNZ $
 66: 021E' DB DC        IN A,(P1_SWPT)
 67: 0220' E6 C0        AND 11000000B
 68: 0222' 0F           RRCA
 69: 0223' 0F           RRCA
 70: 0224' 5F           LD E,A
 71: 0225' DB DD        IN A,(P2_SWPT)
 72: 0227' E6 03        AND 11B
 73: 0229' 0F           RRCA
 74: 022A' 0F           RRCA
 78: 022B' 83           OR E
 79: 022C' 57           LD D,A
 80: 022D' 3E 87        LD A,10000111B
 81: 022F' D3 3F        OUT (PSWC),A
 82: 0231' 06 0D        LD B,13
 83: 0233' 10 FE        DJNZ $
 84: 0235' DB DC        IN A,(P1_SWPT)
 85: 0237' E6 C0        AND 11000000B
 86: 0239' 5F           LD E,A
 87: 023A' DB DD        IN A,(P2_SWPT)
 88: 023C' E6 03        AND 11B
 89: 023E' B3           OR E
 90: 023F' 07           RLCA
 91: 0240' 07           RLCA
 92: 0241' B2           OR D
 93: 0242' ED 44        NEG
 94: 0244' 32 C01A      LD (P2_TRX),A
 95: 0247' TRACK_Y2:
 96: 0247' 3E 07        LD A,00000111B
 97: 0249' D3 3F        OUT (PSWC),A
 98: 024B' 06 19        LD B,25
 99: 024D' 10 FE        DJNZ $
100: 024F' DB DC        IN A,(P1_SWPT)
101: 0251' E6 C0        AND 11000000B
102: 0253' 0F           RRCA
103: 0254' 0F           RRCA
104: 0255' 5F           LD E,A
105: 0256' DB DD        IN A,(P2_SWPT)
106: 0258' E6 03        AND 11B
107: 025A' 0F           RRCA
108: 025B' 0F           RRCA
109: 025C' B3           OR E
110: 025D' 57           LD D,A
111: 025E' 3E 87        LD A,10000111B
112: 0260' D3 3F        OUT (PSWC),A
113: 0262' 06 0D        LD B,13
114: 0264' 10 FE        DJNZ $
115: 0266' DB DC        IN A,(P1_SWPT)
116: 0268' E6 C0        AND 11000000B
117: 026A' 5E           LD E,A
118: 026B' DB DD        IN A,(P2_SWPT)
119: 026D' E6 03        AND 11B
120: 026F' B3           OR E
121: 0270' 07           RLCA
122: 0271' 07           RLCA
123: 0272' B2           OR D
124: 0273' ED 44        NEG
125: 0275' 32 C01B      LD (P2_TRY),A
126: 0278' C9           RET
```
