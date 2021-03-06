# auth.get_group_menu_policy function:

```plpgsql
CREATE OR REPLACE FUNCTION auth.get_group_menu_policy(_role_id integer, _office_id integer)
RETURNS TABLE(row_number integer, menu_id integer, app_name text, i18n_key text, menu_name text, allowed boolean, url text, sort integer, icon character varying, parent_menu_id integer)
```
* Schema : [auth](../../schemas/auth.md)
* Function Name : get_group_menu_policy
* Arguments : _role_id integer, _office_id integer
* Owner : frapid_db_user
* Result Type : TABLE(row_number integer, menu_id integer, app_name text, i18n_key text, menu_name text, allowed boolean, url text, sort integer, icon character varying, parent_menu_id integer)
* Description : 


**Source:**
```sql
CREATE OR REPLACE FUNCTION auth.get_group_menu_policy(_role_id integer, _office_id integer)
 RETURNS TABLE(row_number integer, menu_id integer, app_name text, i18n_key text, menu_name text, allowed boolean, url text, sort integer, icon character varying, parent_menu_id integer)
 LANGUAGE plpgsql
AS $function$
BEGIN
    DROP TABLE IF EXISTS _temp_menu;
    CREATE TEMPORARY TABLE _temp_menu
    (
        row_number                      SERIAL,
        menu_id                         integer,
        app_name                        text,
		i18n_key						text,
        menu_name                       text,
        allowed                         boolean,
        url                             text,
        sort                            integer,
        icon                            national character varying(100),
        parent_menu_id                  integer
    ) ON COMMIT DROP;

    INSERT INTO _temp_menu(menu_id)
    SELECT core.menus.menu_id
    FROM core.menus
    ORDER BY core.menus.app_name, core.menus.sort, core.menus.menu_id;

    --GROUP POLICY
    UPDATE _temp_menu
    SET allowed = true
    FROM  auth.group_menu_access_policy
    WHERE auth.group_menu_access_policy.menu_id = _temp_menu.menu_id
    AND office_id = _office_id
    AND role_id = _role_id;
   
    
    UPDATE _temp_menu
    SET
        app_name        = core.menus.app_name,
		i18n_key		= core.menus.i18n_key,
        menu_name       = core.menus.menu_name,
        url             = core.menus.url,
        sort            = core.menus.sort,
        icon            = core.menus.icon,
        parent_menu_id  = core.menus.parent_menu_id
    FROM core.menus
    WHERE core.menus.menu_id = _temp_menu.menu_id;    

    RETURN QUERY
    SELECT * FROM _temp_menu
    ORDER BY app_name, sort, menu_id;
END
$function$

```

### Related Contents
* [Schema List](../../schemas.md)
* [Function List](../../functions.md)
* [Trigger List](../../triggers.md)
* [Table of Contents](../../README.md)

