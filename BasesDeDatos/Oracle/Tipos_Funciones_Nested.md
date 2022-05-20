# Oracle Tipus, Funcions de tipus, Herencia i Taules niuades

Creem l'objecte **Votant** amb una funcio que retornarà si ha votat o no

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


Ara crearem una taula amb el tipus **votants** que hem creat anteriorment

```sql
create table votants_taula of votant_typ;

-- Inserirem dades a la taula utilitzant el tipus, ja que és una taula de tipus objecte
INSERT INTO votants_taula values (votant_typ('11111111', 'juan', 'perez', 0));
INSERT INTO votants_taula values (votant_typ('11111112', 'jose', 'maria', 1));
INSERT INTO votants_taula values (votant_typ('11111113', 'marcos', 'garcia', 1));
```


SELECTS amb crides de funcio | **Taula objecte**

```sql
-- es una select normal pero a l'hora de cridar la funcio hem de cridar a l'atribut amb el "diminutiu" de la taula "votants_taula v".
select dni, nom, cognoms, v.jaHeVotat() from votants_taula v;

-- Si fem un update podem tornar a fer el select i veurem com ja ha canviat el seu vot
update votants_taula set votat=0 where nom = 'pinlin';
```


CREAR TAULA AMB **REFERENCIES**,  **INSERTS** I **CRIDAR-LOS**

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


COLECCIONS I TAULES NIUADES

