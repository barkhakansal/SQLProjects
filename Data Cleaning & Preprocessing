/*

Cleaning Data in SQL Queries

*/

select * from NashvilleHousing..Housing

--standardize date format

select saledate from NashvilleHousing..Housing

alter table Housing
add SaleDateConverted date

update Housing 
set SaleDateConverted = convert(date,saledate)

select saledate, SaleDateConverted from NashvilleHousing..Housing

-------------------------------------------------------------------------------------------------
--populating property address data

select parcelid,PropertyAddress from NashvilleHousing..Housing
--where PropertyAddress is null

select a.ParcelID, b.ParcelID,a.PropertyAddress, b.PropertyAddress, isnull(a.propertyaddress,b.PropertyAddress) from NashvilleHousing..Housing a
join NashvilleHousing..Housing b
on a.ParcelID = b.ParcelID
and a.[UniqueID ] <> b.[UniqueID ]
where a.PropertyAddress is null

update a
set a.propertyaddress = isnull(a.propertyaddress,b.PropertyAddress)
from NashvilleHousing..Housing a
join NashvilleHousing..Housing b
on a.ParcelID = b.ParcelID
and a.[UniqueID ] <> b.[UniqueID ]
where a.PropertyAddress is null

select parcelid from NashvilleHousing..Housing
where PropertyAddress is null

----------------------------------------------------------------------------------------------------
--Breaking out property address into individual columns(Address, City, State)

select propertyaddress,SUBSTRING(propertyaddress,1,CHARINDEX(',', PropertyAddress) -1) as Address,
SUBSTRING(propertyaddress, CHARINDEX(',',propertyaddress) +1, LEN(propertyaddress)) as City from NashvilleHousing..Housing

alter table housing
add Address nvarchar(255), City nvarchar(255)

update NashvilleHousing..Housing
set Address = SUBSTRING(propertyaddress,1,CHARINDEX(',', PropertyAddress) -1)

update NashvilleHousing..Housing
set City = SUBSTRING(propertyaddress, CHARINDEX(',',propertyaddress) +1, LEN(propertyaddress))

select * from NashvilleHousing..Housing

----------------------------------------------------------------------------------------------------
--Breaking out owner address into individual columns(Address, City, State)

select owneraddress from NashvilleHousing..housing

select 
PARSENAME(replace(owneraddress, ',' , '.'),3),
PARSENAME(replace(owneraddress, ',' , '.'),2),
PARSENAME(replace(owneraddress, ',' , '.'),1)
from NashvilleHousing..Housing

alter table housing
add OwnerSplitAddress nvarchar(255), OwnerSplitCity nvarchar(255),OwnerSplitState nvarchar(255)

update NashvilleHousing..Housing
set OwnerSplitAddress = PARSENAME(replace(owneraddress, ',' , '.'),3)

update NashvilleHousing..Housing
set OwnerSplitCity = PARSENAME(replace(owneraddress, ',' , '.'),2)

update NashvilleHousing..Housing
set OwnerSplitState = PARSENAME(replace(owneraddress, ',' , '.'),1)


----------------------------------------------------------------------------------------------
-- change Y and N to Yes and No in "Sold as Vacant" field

select distinct(SoldAsVacant), count(SoldAsVacant)
from NashvilleHousing..Housing 
group by SoldAsVacant
order by 2

select SoldAsVacant,
case
	when SoldAsVacant = 'Y' then 'Yes'
	when SoldAsVacant = 'N' then 'No'
	else SoldAsVacant
end
from NashvilleHousing..Housing 

update NashvilleHousing..Housing
set SoldAsVacant = case
	when SoldAsVacant = 'Y' then 'Yes'
	when SoldAsVacant = 'N' then 'No'
	else SoldAsVacant
end

------------------------------------------------------------------------------------------
--Remove duplicates

with RowNumCTE as
(
select * ,
ROW_NUMBER() over (partition by parcelid,propertyaddress,saledate,legalreference,saledate order by uniqueid) row_num
from NashvilleHousing..Housing
)

delete from RowNumCTE
where row_num > 1

----------------------------------------------------------------------------------------------
--delete unused columns

alter table NashvilleHousing..Housing
drop column propertyaddress, owneraddress

select * from NashvilleHousing..Housing


