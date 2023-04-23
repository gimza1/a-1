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





