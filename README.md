# Multimedia App Bounty

I extended the multimedia application and added two features

## Description

1. Search - It creates the ability to search files and get to if faster rather than scrolling or straining the eyes to find them

2. Filter - It creates the ability to separate the files according to their file type (videos, audios, images, documents, etc). And even newer file types in future would automatically be added to the filter options

## Why I Chose These Features

### Search Feature

With the new search feature, users are able to find the particular file they would want by it's name. As the number of files increase, this because a crucial feature as they would no need to scroll the entire length of the page (also which keep the attention of the eye) to find their files. They can just insert the file name in the query and find them

### Filter Feature

This also is a very important feature of any file managing application. Users would be able to seperate file by their file type. This enhances user experience by allowing users quickly locate and access specific file types


## How The Code Works

### Search

I built the search function in the following steps

1. Initialize a search state using react's `useState` hook

```javascript
const [search, setSearch] = useState('')
```

2. Create a search bar with a descriptive label and placeholder next to the delete component button.

3. Style the search bar to look similar to the other buttons (background, text color, paadding), and use the style on the component

```jsx
<div style={style.search}>
....
</div>
```

4. Make the search bar a controlled input by giving it the value of the search state (which at the moment is empty (''))

```jsx
<input type="search" value={search}>
```

5. Add an `onChange` event that modifies the `search` state whenever this search input changes

```jsx
<input type="search" value={search} onChange={(e) => setSearch(e.target.value)}>
```

6. Create a function that takes a string and searches through the 'data' array and only return files whose items names are contained in the search string

```jsx
function searchByName(search, data){
  //loops through the data and filters out files whose item.name does not have the search string contained in it
  //return filteredDataArray
}
```

7. Create a `useEffect` react hook that modifies the files shown on the screen by calling the `setMyFiles` function whenever the search value changes

```jsx
useEffect(() => {
  // Sets the displayed files to the result of the searchByName function i.e filteredDataArray
  setMyFiles(searchByName(search, data))
}, [search])
```

And that's it, the search functionality now works as expected. Typing and erasing should trigger the search whenever you do it

### Filter

I implemented the filter by type function in the following steps

1. Create an array made up of all the current types of files in the `data`. For this, I used the javascript `Set`.

```jsx
const types = [...new Set(data.map((item) => item.type))];
```

Doing it this way rather than manually ensures that even if a new file type is added in the data, it works automatically

3. Create a new state for the currently displayed filetype

```jsx
const [fileType, setFileType] = useState('default')
```
I have also chosen not use an empty string for the intial state

3. Create a `Select` HTML element with on `default` option and loop through the `types` array to get all the other options. Also make the `Select` element a controlled input which takes it's value fron the `fileType` state

```jsx
  <select value={fileType}>
{
  types.map((type, index) => <option key={index} value={type}>{type}s</option>)
}
</select>
```


4. Create an `onChange` event handler that modifies the `fileType` whenever the select element is changed

```jsx
<select value={fileType} onChange={(e) => setFileType(e.target.value)}>
```

5. Create the function to change that takes a file type and the data array and returns a filtered array for the file type

```jsx
function filterByType(fileType, data){
  //loop through the data array and removes items whose type does not match the passed `type`
  //return filteredByTypeArray
}
```

6. Create a `useEffect` hook to run everytime the `fileType` state changes and runs the `filterByType` function

```jsx
useEffect(() => {
  setMyFiles(filterByType(fileType, data))
}, [fileType])
```

And that's it, we're able to now filter our data by the file types (videos, audios, documents, images, and any more added)


## Refactor

In the current state, there are a few things in a normal file manager which works a bit differently

1. The search functionality occurs on each key stroke. The would affect performance in the long run
2. The file type filter and the search do not work together i.e if you search `bo` and it shows files with names having `bo` and you want to filter these returned results. It fails because if you then filter, it resets the `search` results to the original before filter. This intuitively is not how normal file manager system works


### Solutions

1. To solve the search on every key stroke, I made a `useDebounce` hook that only changes it's value if some amount of time is passed (i.e the user must have stopped typing). I have set the time to 500ms and also changed the `useEffect` of the search function to run when `debouncedSearch` changes rather than when the `searchChanges`

```jsx
const debouncedSearch = useDebounce(search)
useEffect(() => {
  // Sets the displayed files to the result of the searchByName function i.e filteredDataArray
  setMyFiles(searchByName(debouncedSearch, data))
}, [debouncedSearch])
```

2. To make the search and the filter work together, we have to refactor a few more things

- Change the `searchByName` function to return a boolean rather than do the work of search and filtering the whole array

```jsx
function searchByName(item, debouncedSearch) {
  return item.name.includes(debouncedSearch)
}
```

- Change `filterByType` to also return a boolean rather do the entire filter of the data

```jsx
function filterByType(item, fileType){
  return item.type === fileType
}
```

- Create an `aggregateQuery` function that actually loops through the array and return only elements that are true through both the search and the filter stages

```jsx
function aggregateQuery(searchString, fileType){
//loop through the data array
// check for items that are true after searchByName and filterByType functions
//return a new array with these new items
}
```

- Remove all the other `useEffect` hooks and create a single one that runs this `aggregateQuery` function and sets `myFiles` to the it's result. It would also run whenever `debouncedSearch` and `fileType` changes


```jsx
useEffect(() => {
  setMyFiles(aggregateQuery(debouncedSearch, fileType))
}, [debouncedSearch, fileType])
```

And after these refactors, we have a working search function , filter function and they both are working smoothly together
