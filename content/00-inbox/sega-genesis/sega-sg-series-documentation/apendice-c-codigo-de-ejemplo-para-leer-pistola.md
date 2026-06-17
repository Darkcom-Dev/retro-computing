# Manual de Referencia de Hardware de Sega Mk3 (Rev1) - Páginas 32 a 36

## Apéndice C - Código de Ejemplo para Leer la Pistola

```assembly
 1:
 2:
 3: CDATA  EQU 0C100H    ; Trabajo de Estado de la Pistola (1)
 4: HVCNT  EQU CDATA+1   ; Contador de Guardado en RAM (1)
 5: HVDATA EQU HVCNT+1   ; Trabajo de Guardado de Contadores V.H (40H)
 6: GCHKCT EQU HVDATA+040H ; (1)
 7: GCHKWK EQU GCHKCT+1  ; (10)
 8: SHOOTF EQU GCHKWK+10 ; Bandera de Disparo de la Pistola (1)
 9: HPOSI  EQU SHOOTF+1  ; Datos de Posición H (1)
 10: VPOSI EQU HPOSI+1   ; Datos de Posición V (1)
 11: SWDATA EQU VPOSI+1  ; Datos de Interruptores (2)
 12:
 13:
 14:
 15:
 16:
 17: ;=======================================================;
 18: ;                                                       ;
 19: ;  ****** BÚSQUEDA DE DIRECCIÓN DE DISPARO ******       ;
 20: ;                                                       ;
 21: ;=======================================================;
 22:
 23: GUNS::
 24:         LD HL,HVCNT
 25:         LD DE,HVDATA
 26:         LD C,32
 27: GUNS1::
 28:         LD A,(CDATA)     ; Trabajo de Bandera de Datos de Color
 29:         DEC A
 30:         RET NZ
 31: GUNSPP:
 32:         IN A,(0DDH)
 33:         AND 040H         ; bit 6, un pulso del Jugador 1
 34:         JP NZ,GUNSPP     ; esperar a que esté en bajo
 35:         LD A,(HL)
 36:         CP C
 37:         JR NC,GUNSPP
 38:         INC (HL)         ; Incrementar Contador
 39:         IN A,(07FH)      ; Lectura de Contador H
 40:         LD (DE),A        ; Guardar en RAM
 41:         INC DE
 42:         IN A,(07EH)      ; Lectura de Contador V
 43:         LD (DE),A        ; Guardar en RAM
 44:         INC DE
 45: GUNS2:
 46:         IN A,(0DDH)
 47:         AND 040H         ; bit 6
 48:         JP Z,GUNS2
 49:         JP GUNSPP
 50:
 51: ;=======================================================;
 52: ;      ***** VERIFICACIÓN DE DATOS V y H *****          ;
 53: ;=======================================================;
 54: GUNCHK::
 55:         LD A,(SHOOTF)    ; Bandera de Disparo de Pistola
 56:         OR A
 57:         RET Z
 58:         XOR A
 59:         LD (SHOOTF),A
 60: GUNCHKTI:
 61:         LD A,(HVCNT)     ; Contador V y H
 62:         CP 5
 63:         RET C
 64: ;
 65:         DEC A
 66:         LD B,A
 67:         LD HL,HVDATA+1
 68:         LD E,0H
 69: GUNNCH:
 70:         LD A,(HL)        ; Contador V
 71:         INC E
 72:         LD C,A
 73:         INC HL
 74:         INC HL
 75:         XOR A
 76:         LD A,(HL)
 77:         SUB C
 78:         CP 3
 79:         CALL NC,GCWKL2
 80:         DJNZ GUNNCH
 81:         CALL GCHKL2
 82: ;
 83:         CALL CHGN
 84: ;
 85:         LD A,(HL)        ; LECTURA DE CONTADOR H
 86:         CP 0A0H
 87:         RET NC
 88:         AND A,A          ; FLAG DE ACARREO = LIMPIAR
 89:         SUB 016H         ; A=A-14H
 90: ;
 91: ;SUB 016H-004H : ¿¿ Derecha ?? ?????
 92: ;SUB 016H+004H : ¿¿ Izquierda ?? ?????
 93: ;
 94:         SLA A            ; A=A*2
 95:         LD B,A
 96:         REPT 5
 97:         RRA
 98:         ENDM
 99:         AND A,07H        ; FLAG DE ACARREO = LIMPIAR
100:         ADD B            ; A=A+A/16
101:         LD (HPOSI),A
102:         INC HL
103:         LD A,(HL)        ; Lectura de Contador V
104:         AND A,A          ; FLAG DE ACARREO = LIMPIAR
105: ;       SUB 018H         ; A=A-18H
106:         SUB 001H
107:         LD (VPOSI),A
108: ; * Configuración de Datos de Color * ;
109:         XOR A
110:         OUT (0BFH),A
111:         LD A,0C0H
112:         OUT (0BFH),A
113:         LD BC,020BEH
114:         LD HL,COLORTBL   ; Tabla de Datos de Color
115:         OTIR
116:         RET
117: COLORTBL:
118:         DEFB 000H,000H,000H,000H,000H,000H,000H,000H
119:         DEFB 000H,000H,000H,000H,000H,000H,000H,000H
120:         DEFB 000H,000H,000H,000H,000H,000H,000H,000H
121:         DEFB 000H,000H,000H,000H,000H,000H,000H,000H
122:
123:
124: 125:
126:
127: GCHKL2:
128:         PUSH HL
129:         LD HL,GCHKCT
130:         LD A,(HL)
131:         CP 5
132:         JP NC,GCHL1
133:         INC (HL)
134:         LD HL,GCHKWK
135:         PUSH DE
136:         LD D,0
137:         LD E,A
138:         ADD HL,DE
139:         POP DE
140:         LD (HL),E
141: GCHL1:
142:         POP HL
143:         LD E,0H
144:         RET
145: ;
146:
147: CHGGN:
148:         LD A,(HL)
149:         JP CHGN4
150: ;
151: CHGN:
152:         LD HL,GCHKWK
153:         LD C,0
154:         LD A,(GCHKCT)
155:         CP 1
156:         JP Z,CHGGN
157:         LD B,A
158: CHGN1:
159:         LD A,(HL)
160: CHGN2:
161:         INC HL
162:         CP (HL)
163:         JP C,CHGN3
164:         DJNZ CHGN2
165:         JP CHGN4
166: CHGN3:
167:         INC C
168:         DJNZ CHGN1
169: CHGN4:
170:         RRCA
171:         AND 07FH
172:         LD B,C
173:         LD C,A
174:         LD A,B
175:         OR A
176:         JP Z,CHGN6
177:         LD HL,GCHKWK
178:         XOR A
179: CHGN5:
180:         ADD A,(HL)
181:         DJNZ CHGN5
182:         ADD A,C
183: CHGN7:
184:         ADD A,A
185:         LD C,A
186:         LD B,0
187:         LD HL,HVDATA
188:         ADD HL,BC
189:         RET
190: CHGN6:
191:         LD A,C
192:         JP CHGN7
193:
194:
195: 197: ;=======================================================;
198: ;                                                       ;
199: ;  ***** VERIFICACIÓN DE PISTOLA *****                  ;
200: ;                                                       ;
201: ;=======================================================;
202: GUNINT::
203:         LD HL,CDATA      ; Trabajo de Estados de Destello
204:         LD A,(HL)
205:         LD (HL),2        ;
206:         DEC A            ; CP 1
207:         RET Z            ; (CDATA)=1 ---> (CDATA)=2 , RET
208:         LD (HL),0        ;
209:         DEC A            ; CP 2
210:         RET Z            ; (CDATA)=2 ---> (CDATA)=0 , RET
211: ;
212:         CALL SWSET
213:         AND 010H         ; bit 4
214:         RET Z            ; (CDATA)=0 , TRI.=APAGADO ---> RET
215: ;
216:         XOR A
217:         LD (HVCNT),A
218:         LD A,1           ; A.res = 1
219:         LD (CDATA),A     ; (CDATA)=0 , TRI.=ENCENDIDO ---> (CDATA)=1 , RET
220:         LD (SHOOTF),A    ; Flag de Disparo de Pistola Activado
221: ; * Destello de Pantalla *
222:         XOR A
223:         OUT (0BFH),A
224:         LD A,0C0H
225:         OUT (0BFH),A
226:         LD A,03FH
227:         OUT (0BEH),A
228: ; * Limpieza de RAM *
229:         LD HL,HVCNT
230:         LD DE,HVCNT+1
231:         LD BC,001H+040H+001H+10
232:         LD (HL),0
233:         LDIR
234:         RET
235:
239: ;=======================================================;
240: ;                                                       ;
244: ;  1.0.1.0 ahora                                        ;
246: ;  ------- CPL                                          ;
247: ;  0.1.0.1 ahora                                        ;
248: ;  1.1.0.0 viejo                                        ;
249: ;  ------- AND                                          ;
250: ;  0.1.0.0                                              ;
251: ;  ------- CPL                                          ;
252: ;  1.0.1.1                                              ;
253: ;=======================================================;
254: SWSET:
255:         IN A,(0DCH)      ;
256:         AND 010H         ; bit 4
257:         LD HL,SWDATA     ; Área de guardado de datos de interruptores
258:         CPL
259:         LD C,A           ; Guardado de datos
260:         XOR (HL)
261:         LD (HL),C        ; Nuevo guardado de datos SW.
262:         INC HL
263:         AND C            ; ACC : cambio '0' --> '1' pero datos
264:         LD (HL),A        ; Guardado de datos
265:         RET
266:
272: ;=======================================================;
273: ;  *** Verificación de Salto de Interrupción ***        ;
274: ;=======================================================;
275: :ORG 038H
276:         PUSH AF
277:         LD A,(CDATA)     ; ¿Pistola Configurada?
278:         DEC A
279:         JR NZ,INSS
280:         EX (SP),HL       ; Cambio de Puntero de Pila
281:         POP HL           ;
282:         LD HL,GUNS1      ;
283:         EX (SP),HL       ;
284:         PUSH AF
285: INSS:
286:         POP AF
287:         JP INT38
```
