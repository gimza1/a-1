RANDOM:

CREATE OR REPLACE PACKAGE customer_pkg AS

  -- CREATE
  PROCEDURE create_customer (
    p_customer_id IN NUMBER,
    p_first_name IN VARCHAR2,
    p_last_name IN VARCHAR2,
    p_email IN VARCHAR2
  );
  
  -- READ
  FUNCTION get_customer (
    p_customer_id IN NUMBER
  ) RETURN SYS_REFCURSOR;
  
  -- UPDATE
  PROCEDURE update_customer (
    p_customer_id IN NUMBER,
    p_first_name IN VARCHAR2,
    p_last_name IN VARCHAR2,
    p_email IN VARCHAR2
  );
  
  -- DELETE
  PROCEDURE delete_customer (
    p_customer_id IN NUMBER
  );
  
END customer_pkg;
/

CREATE OR REPLACE PACKAGE BODY customer_pkg AS

  -- CREATE
  PROCEDURE create_customer (
    p_customer_id IN NUMBER,
    p_first_name IN VARCHAR2,
    p_last_name IN VARCHAR2,
    p_email IN VARCHAR2
  ) AS
  BEGIN
    INSERT INTO customers (
      customer_id,
      first_name,
      last_name,
      email
    ) VALUES (
      p_customer_id,
      p_first_name,
      p_last_name,
      p_email
    );
    COMMIT;
  END;
  
  -- READ
  FUNCTION get_customer (
    p_customer_id IN NUMBER
  ) RETURN SYS_REFCURSOR AS
    v_cursor SYS_REFCURSOR;
  BEGIN
    OPEN v_cursor FOR
      SELECT *
      FROM customers
      WHERE customer_id = p_customer_id;
    RETURN v_cursor;
  END;
  
  -- UPDATE
  PROCEDURE update_customer (
    p_customer_id IN NUMBER,
    p_first_name IN VARCHAR2,
    p_last_name IN VARCHAR2,
    p_email IN VARCHAR2
  ) AS
  BEGIN
    UPDATE customers
    SET first_name = p_first_name,
        last_name = p_last_name,
        email = p_email
    WHERE customer_id = p_customer_id;
    COMMIT;
  END;
  
  -- DELETE
  PROCEDURE delete_customer (
    p_customer_id IN NUMBER
  ) AS
  BEGIN
    DELETE FROM customers
    WHERE customer_id = p_customer_id;
    COMMIT;
  END;
  
END customer_pkg;
/

MUKI PROCEDURE:
CREATE OR REPLACE PROCEDURE get_customers(page_no NUMBER,
page_size NUMBER)
AS
c_customers SYS_REFCURSOR;
c_total_row SYS_REFCURSOR;
BEGIN
-- otvaranje prvog kursora ciji ce rezultujuci set podataka
-- biti vracen procedurom
OPEN c_total_row FOR
SELECT COUNT(*)
FROM customers;

DBMS_SQL.RETURN_RESULT(c_total_row);

-- otvaranje drugog kursora ciji ce rezultujuci set podataka
-- biti vracen procedurom
OPEN c_customers FOR
SELECT customer_id, name
FROM customers
ORDER BY name
OFFSET page_size * (page_no - 1) ROWS
FETCH NEXT page_size ROWS ONLY;

DBMS_SQL.RETURN_RESULT(c_customers);
END;

MUKI FUNCTION:
CREATE OR REPLACE FUNCTION get_total_sales(in_year PLS_INTEGER)
RETURN NUMBER
IS
l_total_sales NUMBER := 0;
BEGIN
-- izvrsavanje upita koji vraca informaciju o ukupnoj prodaji
-- u odredjenoj godini
SELECT SUM(unit_price * quantity)
INTO l_total_sales
FROM order_items
INNER JOIN orders USING (order_id)
WHERE status = 'Shipped'
GROUP BY EXTRACT(YEAR FROM order_date)
HAVING EXTRACT(YEAR FROM order_date) = in_year;

-- funkcija vraca rezultat dobijen upitom
RETURN l_total_sales;
END;

CREATE OR REPLACE NONEDITIONABLE PACKAGE BODY ""."PAC_REGION" AS

FUNCTION kursoriRegiona(pa_Region_ID IN NUMBER,
                        pa_Region_Name IN VARCHAR2,
                        paDodeljeniID OUT NUMBER)RETURN SYS_REFCURSOR AS
                        
     c_Regioni SYS_REFCURSOR;
    
    BEGIN
    IF pa_Region_ID IS NOT NULL THEN
    OPEN c_Regioni FOR
    SELECT *
    FROM Regions
    WHERE Region_ID=pa_Region_ID;
    END IF;
    RETURN c_Regioni;
    END kursoriRegiona;
-------------------------------------------------------------------------------------------------------   
PROCEDURE brisanje(pa_Region_ID IN NUMBER) AS
BEGIN
DELETE
FROM Regions
WHERE Region_ID=pa_Region_ID;
END brisanje;
-------------------------------------------------------------------------------------------------------
FUNCTION brojZavisnihTabela(pa_Region_ID IN NUMBER)RETURN SYS_REFCURSOR AS
c_brojTabela SYS_REFCURSOR;
BEGIN
OPEN c_brojTabela FOR
SELECT COUNT(DISTINCT(Country_ID)) AS BrojDrzava,
       COUNT(DISTINCT(Location_ID)) AS BrojLokacija,
       COUNT(DISTINCT(warehouse_ID)) AS BrojMagacina,
       COUNT(DISTINCT(product_ID)) AS BrojProizvoda
FROM Country LEFT JOIN Locations USING (country_ID)
             LEFT JOIN Warehouses USING (location_ID)
             LEFT JOIN Inventories USING (warehouse_ID)
             
WHERE Countries.pa_Region_ID = Region_ID;
RETURN c_brojTabela;
END;

END brojZavisnihTabela;
----------------------------------------------------------------------------------
PROCEDURE insertUpdate(pa_Region_ID IN NUMBER,
                       pa_Region_Name IN VARCHAR2,
                       paDodeljeniID OUT NUMBER) AS

izuzetak EXCEPTION;
PRAGMA EXCEPTION_INIT(izuzetak, -20001);
CURSOR c_Region IS
SELECT Region_Name
FROM Regions
WHERE pa_Region_ID = Region_ID
FOR UPDATE OF Region_Name;
BEGIN
IF pa_Region_ID<>0 THEN
   IF broj_slogova(q'[Regions WHERE UPPER(Region_Name)=']' || UPPER(pa_Region_Name) || q'[' AND region_ID<>']' || pa_Region_ID)>0 THEN
      RAISE Izuzetak;
    END IF;
   FOR r_Region IN c_Region
   LOOP
   UPDATE Regions
   SET region_Name=pa_Region_Name
   WHERE CURRENT OF c_Region;
   END LOOP;
   COMMIT;
ELSE
    IF broj_slogova(q'[Regions WHERE UPPER(Region_Name=')]' || UPPER(pa_Region_Name) || q'[']') THEN
       RAISE Izuzetak;
    END IF;
    
    INSERT INTO Regions(Region_ID, Region_NAME)
    VALUES (DEFAULTS, pa_region_ID)
    RETURNING Region_ID INTO paDodeljeniID;
    COMMIT;
END IF;
END insertUpdate;

END pac_Region; 
