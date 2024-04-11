# Bug 1: Select dropdown doesn't scroll with rest of the page
### Cause: 
`fixed` position property will cause the dropbar to stay stuck on the page. If we remove it, it will stay to the parent element.

### Fixes:
- Removed `position: fixed;` from `.RampInputSelect--dropdown-container` in `index.css`

# Bug 2: Approve checkbox not working
### Cause: 
`display` property in index.css for the input element is set to `none` meaning it loses its functionality. We need to overall the input over the label element.

### Fixes:
- Added `position: relative` to `.RampInputCheckbox--container` in `index.css`
- Removed `dispay: none` from `.RampInputCheckbox--input` from `index.css`
- Overlayed the checkbox ontop of the checkbox label with the `absolute` position and adjusted the size to match up with the label 

# Bug 3: Cannot select All Employees after selecting an employee
### Cause:
We don't do a check before calling `loadTransactionByEmployee(id)`. Instead we should call `loadAllTransactions()` when we filter by `All Employees`

### Fixes:
- Had to add an if check to see if the ID is empty: `""` in `App.tsx`
- ```if (newValue.id === "") await loadAllTransactions();
            else await loadTransactionsByEmployee(newValue.id);```

# Bug 4: Clicking on View More button not showing correct data
### Cause:
We do not concatenate the data from the new page, only showing the new results.

### Fixes:
- Combined previous paginated responses data with the current one in `usePaginatedTransactions.ts`
```
let combinedData = previousResponse.data.concat(response.data);
return { data: combinedData, nextPage: response.nextPage };
```

# Bug 5: Employees filter not available during loading more data
### Cause:
We have not set `isLoading` state to false after loading employees even though the request is complete.

### Fixes: 
- Move the `setIsLoading(false)` line to after `employeeUtils` fetches all the employees in `App.tsx`
- The filter dropbar is dependent on that state to display its results so we have to set loading to false after it retrieves until instead of waiting for the paginated results
```
setIsLoading(true);
transactionsByEmployeeUtils.invalidateData();

await employeeUtils.fetchAll();
setIsLoading(false);

await paginatedTransactionsUtils.fetchAll();
```

# Bug 6: View more button not working as expected
### Cause:
We don't check if we are loading paginated transactions when rendering the `View More` button and we don't check if there is a next page

### Fix: 
- Add 2 additional checks before rendering the button
- paginatedTransactions will be null when we are loading by employee
- paginatedTransactions.nextPage will be null when there are no more transactions
```
{
    transactions !== null &&
    paginatedTransactions !== null &&
    paginatedTransactions.nextPage !== null && 
    (
        <button
            className="RampButton"
            disabled={paginatedTransactionsUtils.loading}
            onClick={async () => {
                  await loadAllTransactions();
            }}
        >
        View More
        </button>
    )}
```

# Bug 7: Approving a transaction won't persist the new value
### Cause: 
The web page was caching the results and rendering using old data

### Fix: 
- Use the fetch without cache function instead in `usePaginatedTransactions.ts` and `useTransactionsByEmployees.ts`

```
const fetchById = useCallback(
    async (employeeId: string) => {
        const data = await fetchWithoutCache<
        Transaction[],
        RequestByEmployeeParams
        >("transactionsByEmployee", {
        employeeId,
        });

        setTransactionsByEmployee(data);
    },
    [fetchWithoutCache]
);
```

```
const fetchAll = useCallback(async () => {
    const response = await fetchWithoutCache<
        PaginatedResponse<Transaction[]>,
        PaginatedRequestParams
    >("paginatedTransactions", {
        page: paginatedTransactions === null ? 0 : paginatedTransactions.nextPage,
    });

    setPaginatedTransactions((previousResponse) => {
        if (response === null || previousResponse === null) {
        return response;
        }

        let combinedData = previousResponse.data.concat(response.data);
        return { data: combinedData, nextPage: response.nextPage };
    });
}, [fetchWithoutCache, paginatedTransactions]);
```

