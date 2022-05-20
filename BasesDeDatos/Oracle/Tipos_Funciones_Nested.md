# Oracle Tipus, Funcions de tipus, Herencia i Taules niuades
  
## Creem l'objecte **Votant** amb una funcio que retornarà si ha votat o no
  
```sql
create type votant_typ as object (
    dni varchar2(9),
    nom varchar2(20),
    cognoms varchar2(40),
    votat number,
    member function jaHeVotat return varchar2
);
```

Ara editarem la funcio **jaHeVotat** i farem que miri si el seu vot es 1 o 0

```sql
create or replace type body votant_typ as -- aqui posem el nom del tipus.
member function jaHeVotat return varchar2 is -- i aqui el nom de la funcio.
    begin
        if votat = 1 then
            return 'Ja ha votat'; -- si el seu atriubut votat es 1 retornara que ha votat.
        end if;
        return 'No ha votat encara'; -- si el seu atriubut votat es 0 retornara que no ha votat.
    end;
end;
```
  
  
  
## Ara crearem una taula amb el tipus **votants** que hem creat anteriorment

```sql
create table votants_taula of votant_typ;

-- Inserirem dades a la taula utilitzant el tipus, ja que és una taula de tipus objecte
INSERT INTO votants_taula values (votant_typ('11111111', 'juan', 'perez', 0));
INSERT INTO votants_taula values (votant_typ('11111112', 'jose', 'maria', 1));
INSERT INTO votants_taula values (votant_typ('11111113', 'marcos', 'garcia', 1));
```
  
  

## SELECTS amb crides de funcio | **Taula objecte**

```sql
-- es una select normal pero a l'hora de cridar la funcio hem de cridar a l'atribut amb el "diminutiu" de la taula "votants_taula v".
select dni, nom, cognoms, v.jaHeVotat() from votants_taula v;

-- Si fem un update podem tornar a fer el select i veurem com ja ha canviat el seu vot
update votants_taula set votat=0 where nom = 'pinlin';
```
  
  

## CREAR TAULA AMB **REFERENCIES**,  **INSERTS** I **CRIDAR-LOS**

```sql
  -- Creem una taula amb un atribut que es tipus votant_typ, i fara referencia a un valor de la taula votants_taula 
  --(una persona existent a la taula votants_taula)
create table seccions_taula(
    persona ref votant_typ references votants_taula,
    codiPostal number,
    colegi varchar2(40),
    mesaElectoral number
);
```

Ara el que fem es un **insert**, pero per aixo hem **d'agafar** un objecte de la taula votants, ja que com es una referencia
li hem de pasar un objecte existent

```sql
declare
votant_varRef ref votant_typ; --Crearem una variable que sera una referencia del tipus votant_typ
begin
--Guardarem el resultat de la select, que en aquest cas es una referencia "ref(v)" a la variable que hem creat
SELECT ref(v) into votant_varRef from votants_taula v where nom = 'andre';

-- Farem el insert i li pasarem la referencia (variable) en l'atribut de la taula que era una referencia de la taula votants_taula
INSERT INTO seccions_taula values (votant_varRef, 17469, 'cendrassos', 1);

--Es poden fer més d'un insert en un mateix bloc anonim utilitzant la mateixa variable fent
  -- SELECT -> guardo variable
  -- INSERT -> a taula
  -- SELECT -> guardo variable
  -- INSERT -> a taula

end;
```

Per accedir als **atributs** del atribut **persona** de la taula seccions utilitzem deref(atribut).atributdeobjecte, i per les seves funcions el mateix pero acaba en ()

```sql
-- El que fem aqui es agafar tots els atributs de les persones de la taula seccions_taula i mirar si han votat o no amb la seva funcio
select deref(persona).dni, deref(persona).nom, deref(persona).cognoms, deref(persona).jaHeVotat() from seccions_taula;
```
  
  

## COLECCIONS I TAULES NIUADES
  
Primer de tot creem el tipus membre per poder guardar membres
  
```sql
CREATE TYPE membre_mesa_typ as object (
    dni varchar2(9),
    nom varchar2(20),
    cognoms varchar2(40),
    carreg varchar2(20)
);
```
  
Crearem una **"taula"** de tipus membre mesa type, que guardara mes d'un membre, per ferla **niuada** mes tard
```sql
create type conjunt_membres_typ as table of membre_mesa_typ;
```
  
Ara crearem una taula **mesa_electoral** amb un atribut que sera un tipus taula que podra guardar més d'un membre a l'hora  
```sql
create table mesa_electoral(
colegi varchar2(40),
numeromesa number,
membres_mesa conjunt_membres_typ)
nested table membres_mesa store as membres_mesa_nt; -- <- aqui fem la taula niuada amb el nom de l'atribut, i el segon nom es el mateix mes _nt que es per oracle temes de memoria (no s'utilitza)
```
  
Inserts a la **taula niuada**
  
```sql  
insert into mesa_electoral values ('muntaner', 2, -- Aqui inserim els seus atributs
conjunt_membres_typ( -- Aqui inserim l'atribut de tipus "taula" niuada i a dins posarem els tipus membres que son tipus membres
membre_mesa_typ('11111111', 'lin', 'martinez', 'president'),
membre_mesa_typ('11111111', 'jordi', 'martinez', 'vocal1'),
membre_mesa_typ('11111111', 'andre', 'martinez', 'vocal2')
));
```

L'esquema seria alguna cosa així:
  
  * Mesa_electoral (taula)
    * els seus atributs
    * atribut de tipus taula (conjunt_membres_typ) -> aquest tipus podia guardar multiples membres com hem dit mes a dalt
      * tipus membre (membre_mesa_type)
      * tipus membre (membre_mesa_type)      
      * tipus membre (membre_mesa_type)      

       
Farem un **SELECT** de les taules mesa electoral, per mostrar els seus atributs i mostrar els registres de la taula niuada

```sql
set serveroutput on;
declare
colegi_var varchar2(60);
numeroMesa_var number;
persona conjunt_membres_typ; -- Aquest es un tipus conjunt_membres, on podrem guardar tots els registres per després iterar-los (com un arraylist)
i number;
begin
select colegi, numeroMesa, membres_mesa into colegi_var, numeroMesa_var, persona from mesa_electoral where numeroMesa = 2; -- agafem els atributs de la taula mesa i els seus membres guardant-los a una variable de tipus taula que ara utilitzarem com a array
for i in 1..persona.count loop -- iterarem el arraylist posant el atribut (i) que cada cop agafara un element diferent de la llista i mostrarem les seves dades.
DBMS_OUTPUT.PUT_LINE(colegi_var || numeroMesa_var ||' '||persona(i).dni||' '|| persona(i).nom||' '|| persona(i).carreg);
end loop;
end;       
```