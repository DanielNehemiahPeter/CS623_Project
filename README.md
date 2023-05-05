# CS623_Project

#POSTGRES(Tables Creation)

CREATE TABLE Product(prodid CHAR(10), pname VARCHAR(30), price DECIMAL); 
CREATE TABLE Depot(depid CHAR(10), addr VARCHAR(25), volume INT); 
CREATE TABLE Stock(prodid CHAR(10), depid CHAR(10), quantity FLOAT); 

ALTER TABLE Product add constraint pk_product primary key(prodid); 
ALTER TABLE Product add constraint ck_price check(price>0); 
ALTER TABLE Depot add constraint pk_depot primary key(depid); 
ALTER TABLE Depot add constraint ck_depot_volumne check(volume>0); 
ALTER TABLE Stock add constraint pk_stock primary key(prodid,depid); 
ALTER TABLE Stock add constraint fk_prodid foreign key(prodid) references Product(prodid); 
ALTER TABLE Stock add constraint fk_depid foreign key(depid) references Depot(depid); 
ALTER TABLE Stock add constraint ck_quantity check(quantity<>0); 

INSERT into Product(prodid,pname,price) values('P1','tape',2.5),('P2','tv',250),('P3','vcr',80); 
INSERT into Depot(depid,addr,volume) values('D1','New York',9000),( 'D2','Syracuse',6000),( 'D4','New York',2000); 
INSERT into Stock(prodid,depid,quantity) values('P1','D1',1000),('P1','D2',-100),('P1','D4',1200),('P3','D1',3000),('P3','D4',2000),('P2','D4',1500),('P2','D1',-400),('P2','D2',2000); ;

#JUPYTER NOTEBOOK- Connecting PYTHON to POSTGRES SERVER
pip install psycopg2
pip install tabulate

import psycopg2
from tabulate import tabulate

con = psycopg2.connect(
    host="localhost",
    database="CS623_Project",
    user="postgres",
    password="3303")
    
print(con)

#For isolation: SERIALIZABLE
con.set_isolation_level(3)
#For atomicity
con.autocommit = False

#1. The product p1 is deleted from Product and Stock.
try:
    cur = con.cursor()
    #Begin transaction
    cur.execute("START TRANSACTION;")
 
    #Delete from Stock
    cur.execute("DELETE FROM Stock WHERE prodid = 'p1';")
    
    #Delete from Product
    cur.execute("DELETE FROM Product WHERE prodid = 'p1';")
    
except (Exception, psycopg2.DatabaseError) as err:
    print(err)
    print("Transactions could not be completed so database will be rolled back before start of transactions")
    con.rollback()
finally:
    if con:
        con.commit()
        cur.close
        con.close
        print("PostgreSQL connection is now closed")
