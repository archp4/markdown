# P/L SQL

## Question 1
Create a procedure create a procedure bb_get_status that returns the most recent status (from the BB_BASKETSTATUS table), in English.

Find the largest IDSTATUS for the specified  basket id (IDBASKET)
Return the description of the IDSTAGE as follows:
1 - Submitted and received
2 - Confirmed, processed, sent to shipping
3 -  Shipped
4 - Cancelled
5 - Back-ordered
If an invalid basket id is provided, return “Invalid basked id”
Use if or case to translate the IDSTAGE to English text.
	
To test the code execute the following:

```SQL

set serveroutput on
declare
    v_desc  varchar(100);
begin
    bb_get_status(3, v_desc);
    print_line('Basket 3: ' || v_desc);

    bb_get_status(6, v_desc);
    print_line('Basket 6: ' || v_desc);
end;

```

Procedure Code:

```SQL

create or replace procedure BB_GET_STATUS ( P_BASKET_ID in BB_BASKETSTATUS.IDBASKET%type, 
                                            P_STATUS out varchar2 
                                            ) is
    V_MAX_ID number(2) := 0; -- To Store Max Status ID
    V_STAGE_ID BB_BASKETSTATUS.IDSTAGE%type; -- To string result
begin 
    -- Getting record with max status
    for V_REC in ( select *
                     from BB_BASKETSTATUS 
                    where IDBASKET = P_BASKET_ID
                    order by IDBASKET desc )
    loop
       if ( V_REC.IDSTATUS > V_MAX_ID ) then
           V_MAX_ID := V_REC.IDSTATUS;
           V_STAGE_ID := V_REC.IDSTAGE;
       end if;
    end loop;
    
    -- Returning Stage Detail in English instead of Number Value
    if ( V_STAGE_ID = 1 ) then
        P_STATUS := 'Submitted and received';
    elsif ( V_STAGE_ID = 2 ) then
        P_STATUS := 'Confirmed, processed, sent to shipping';
    elsif ( V_STAGE_ID = 3 ) then
        P_STATUS := 'Shipped';
    elsif ( V_STAGE_ID = 4 ) then
        P_STATUS := 'Cancelled';
    elsif ( V_STAGE_ID = 5 ) then
        P_STATUS := 'Back-ordered';
    else
        P_STATUS := 'Stage Not Found';
    end if;    

exception
    -- if datafouund this block will be excute
    when NO_DATA_FOUND then
        PRINT_LINE('Invalid Basket ID : ' || P_BASKET_ID);

end BB_GET_STATUS;

```


## Question 2

Rewrite bb_get_status as a function bb_fget_status that returns the status description.
Test it by running the following code:


```SQL

create or replace function BB_FGET_STATUS ( P_BASKET_ID BB_BASKETSTATUS.IDBASKET%type) 
                                            RETURN VARCHAR2 is
    V_MAX_ID number(2) := 0; -- To Store Max Status ID
    V_STAGE_ID BB_BASKETSTATUS.IDSTAGE%type; -- To Stage Id for if else ladder 
    P_STATUS VARCHAR(100); -- To string result
begin 
    -- Getting record with max status 
    for V_REC in ( select *
                     from BB_BASKETSTATUS 
                    where IDBASKET = P_BASKET_ID )
    loop
       if ( V_REC.IDSTATUS > V_MAX_ID ) then
           V_MAX_ID := V_REC.IDSTATUS;
           V_STAGE_ID := V_REC.IDSTAGE;
       end if;
    end loop;
    
    -- Returning Stage Detail in English instead of Number Value
    if ( V_STAGE_ID = 1 ) then
        P_STATUS := 'Submitted and received';
    elsif ( V_STAGE_ID = 2 ) then
        P_STATUS := 'Confirmed, processed, sent to shipping';
    elsif ( V_STAGE_ID = 3 ) then
        P_STATUS := 'Shipped';
    elsif ( V_STAGE_ID = 4 ) then
        P_STATUS := 'Cancelled';
    elsif ( V_STAGE_ID = 5 ) then
        P_STATUS := 'Back-ordered';
    else
        P_STATUS := 'Stage Not Found';
    end if;    
    return P_STATUS;
exception
    -- if datafouund this block will be excute
    when NO_DATA_FOUND then
        PRINT_LINE('Invalid Basket ID : ' || P_BASKET_ID);

end BB_FGET_STATUS;


```

## Question 3

Write a function bb_subtotal that returns the total cost of a basket, including a new tax of 5%  by scanning the basket content of a specified basked id.

If an invalid basked is provided, raise the NO_DATA_FOUND exception.	
Function Code:


```SQL

create or replace function BB_SUBTOTAL ( P_BASKET_ID bb_basket.IDBASKET%type) 
                                            RETURN NUMBER is
--  delcaring  variable 
    v_rec bb_basket%rowtype;
    v_temp_total Number(10);
begin 
    -- Fetch Record for required basked id
    select * 
      into v_Rec 
      from bb_basket
     where IDBASKET = P_BASKET_ID;
    -- total without tax
    v_temp_total := ( v_rec.subtotal + v_rec.shipping );
    -- total with tax 5%
    return (v_temp_total + (v_temp_total * 0.05));
exception
    when NO_DATA_FOUND then
        -- If Basket Id not found in Database
        PRINT_LINE('Invalid Basket ID : ' || P_BASKET_ID);

end BB_SUBTOTAL;


```

TEST CODE


```SQL

set serveroutput on
begin
    PRINT_LINE('Basket 3: ' || BB_SUBTOTAL(3)); -- Test for Basket Id 3
    PRINT_LINE('Basket 6: ' || BB_SUBTOTAL(6)); -- Test for Basket Id 6
end;


```
