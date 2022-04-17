# DataBase
## Créer la base de données
```sql
USE master;
GO
IF DB_ID (N'BOUTIQUE') IS NOT NULL
DROP DATABASE BLOODBANK;
GO
CREATE DATABASE BLOODBANK;
GO
```
## Table PRODUITS
```sql
USE [BOUTIQUE]
GO

CREATE TABLE [dbo].[PRODUITS](
	[Id] [int] NOT NULL,
	[Designation] [varchar](254) NOT NULL,
	[Quantite] [int] NOT NULL,
	[Prix] [decimal](18, 2) NOT NULL
) ON [PRIMARY]
GO
```
### Alimenter la table PRODUITS
```sql
USE [BOUTIQUE]
GO
DECLARE @i int = 1
WHILE @i <= 100
BEGIN
  INSERT PRODUITS VALUES(@i, CONCAT('Produit ', @i), 0, 0)
  SET @i = @i +1
END
GO
```
## Bons de Livraison
### Création du schéma "livraisons"
```sql
USE [BOUTIQUE]
GO
CREATE SCHEMA [livraison]
GO
```
### Création de la table livraison.ENTETES
```sql
USE [BOUTIQUE]
GO
CREATE TABLE [livraison].[ENTETES](
	[Numero] [int] NOT NULL,
	[Date] [datetime] NOT NULL,
	[Fournisseur] [varchar](50) NOT NULL,
	[Etat] [int] NOT NULL
) ON [PRIMARY]

GO
```
### Création de la table livraison.PRODUITS
```sql
USE [BOUTIQUE]
GO
CREATE TABLE [livraison].[PRODUITS](
	[NumeroBon] [int] NOT NULL,
	[CodeProduit] [int] NOT NULL,
	[Quantite] [int] NOT NULL,
	[PrixAchat] [decimal](18, 2) NOT NULL
) ON [PRIMARY]
GO
```
### Alimenter les tables livraison.ENTETES et livraison.PRODUITS
```sql
USE [BOUTIQUE]
GO

INSERT livraison.ENTETES VALUES(1, '15/04/2022', 'Fournisseur 1', 0)

declare @i int = 1

while (@i <=20)
begin
	insert livraison.PRODUITS 
	values(1, @i, @i*71%29, @i*71%29*1.65)
	set @i=@i+1
end
```

### Création Procédure stockée pour valider une livraison
```sql
USE [BOUTIQUE]
GO
CREATE PROCEDURE livraison.Valider
	@aNumBon INT
AS BEGIN
IF (SELECT etat FROM livraison.ENTETES WHERE Numero = 1) =1
RETURN 1001

UPDATE dbo.PRODUITS SET
	dbo.PRODUITS.Quantite = dbo.PRODUITS.Quantite+PL.Quantite,
	dbo.PRODUITS.Prix = PL.PrixAchat
FROM livraison.PRODUITS AS PL
WHERE dbo.PRODUITS.Id = PL.CodeProduit AND PL.NumeroBon = @aNumBon

UPDATE livraison.ENTETES
SET Etat=1
WHERE Numero=@aNumBon

RETURN 0
END

```

## Table ERRORS
```sql
USE [BOUTIQUE]
GO

CREATE TABLE [dbo].[ERRORS](
	[Code] [int] NOT NULL,
	[ErrorName] [varchar](50) NOT NULL,
 CONSTRAINT [PK_ERRORS] PRIMARY KEY CLUSTERED (	[Code] ASC)
) ON [PRIMARY]
GO

INSERT ERRORS VALUES (1001, 'Livraison déjà validée')
```

## Execution de la procédure stockées
```sql
USE [BOUTIQUE]
GO

CREATE TABLE [dbo].[ERRORS](
	[Code] [int] NOT NULL,
	[ErrorName] [varchar](50) NOT NULL,
 CONSTRAINT [PK_ERRORS] PRIMARY KEY CLUSTERED (	[Code] ASC)
) ON [PRIMARY]
GO

INSERT ERRORS VALUES (1001, 'Livraison déjà validée')
```sql
USE [BOUTIQUE]
GO

DECLARE	@return_value int

EXEC	@return_value = [livraison].[Valider]
		@aNumBon = 1

SELECT	'Return Value' = 
case @return_value 
	when 0 then 'Succes' 
	else (select ErrorName from ERRORS where Code =@return_value) 
	end

GO

```
