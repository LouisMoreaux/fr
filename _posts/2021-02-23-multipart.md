---
title: "Oracle APEX 20.2 et multipart/form-data"
last_modified_at: 2021-02-23T23:00:00+01:00
categories:
  - Oracle APEX
tags:
  - multipart/form-data
  - APEX
  - web service
---

Avant APEX 20.2, le package APEX_WEB_SERVICE ne supportait pas le multipart/form-data. Si comme moi, vous deviez atteindre un service web qui n'acceptait que ce type de corps de requête, vous deviez utiliser le package ULT_HTTP et créer le corps par vous-même. Nick Buytaert a fait un bon [post](https://apexplained.wordpress.com/2016/03/21/utl_http-and-a-multipartform-data-request-body/) à ce sujet sur son blog (qui m'a beaucoup aidé).

Mais grâce à l'équipe APEX, le package [APEX_WEB_SERVICE](https://docs.oracle.com/en/database/oracle/application-express/20.2/aeapi/APEX_WEB_SERVICE.html) dans la version 20.2 introduit le support du multipart/form-data. La création de ce type de corps de requête n'a jamais été aussi simple.

```sql
declare
    l_multipart apex_web_service.t_multipart_parts;
    l_blob blob;
    l_body blob;
begin
    select blob_column 
    into l_blob
    from my_file_table
    where id = 1;
    
    apex_web_service.append_to_multipart(
        p_multipart    => l_multipart,
        p_name         => 'file',
        p_content_type => 'application/octet-stream',
        p_body_blob    => l_blob
    );
        
    l_body := apex_web_service.generate_request_body(
        p_multipart => l_multipart
    );
    
    l_response := apex_web_service.make_rest_request(
    	p_url => 'https://...',
        p_http_method => 'POST',
        p_body_blob => l_body
    );
end;
```
Le contenu  envoyé avec un simple fichier texte est 

```
--A8BEA21E96C5B912
Content-Disposition: form-data; name="file"
Content-Type: application/octet-stream

This is a simple text file.
--A8BEA21E96C5B912--
```