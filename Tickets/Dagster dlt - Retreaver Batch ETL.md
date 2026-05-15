>[AN-576](https://ahealthmarketplace.atlassian.net/jira/software/c/projects/AN/boards/52?assignee=712020:c49b8b28-f522-4d9f-809f-e30b6d8317ef&selectedIssue=AN-576)

- Authentication uses a simple key param on each request.
- use V2
- Use Page number pagination. 
- Use the new components structure. Create the dlt pipeline under `defs>Retreaver Batch` 

#### Issues:
1. **Pagination Issue:** 
   While Incrementally loading, When paginating through API results while new records are being created, the pagination becomes unstable. Records can shift between pages, causing you to either miss records or process duplicates.
   **Enumeration Solution:** Use explicit time boundaries (`created_at_start/end`) with consistent ascending sort (`order=asc`) to create a fixed, bounded dataset that won't change during pagination. This ensures you process every record exactly once, regardless of new data being created while you're extracting.
   The enumeration approach turns an unstable, moving target into a stable snapshot you can reliably paginate through.



expected a list of dictionary


custom paginator:
- modify update _state func_ in subclass HeaderLinkPaginator
- if next>last then update → set hasNextPage to false
- for HeaderLinkPaginator
- What is super?
#### Un-nesting JSON response
For write disposition = merge to work, dlt looks for the primary key in the key section of the response. currently the response is a list of call objects. we want to transform it to a list of call response. ie un-nest from `[{"call": {...}}, {"call": {...}}]` to `[{...}, {...}]`.

## Write dispositions:
1. incremental merge → **calls** since transactional 
2. full refresh
	1. target → 38
	2. affiliates → 130
	3. campaigns → 18

## Update:
1. Numbers and Numbers Pools  not working
2. companies just 1 record synergy
3. contacts would be too long. do we even need it?
4. target group empty
5. 
## Challenges:
### 1. Can’t pass multiple arguments to the map function in processing_steps:
- DLT's map function expects:
	1. A function object (not a function call result)
	2. That function should accept exactly one parameter: the record to transform
	3. DLT will call this function for each record in the data stream
- The problem is that the custom `extract_call_data` map function expects two arguments (record and resource_key), but the map function by dlt can only pass the record.
  We need to create a factory function that returns a closure with the resource_key pre-configured.
- **Factory Function**: A function that creates and returns other functions.
- **Closure**: A function that captures variables from its outer scope. The inner `extract_call_data` function is the closure because it captures the `resource_key` variable from the outer function's scope.
- Why adding additional argument won’t work:
```python
	# This CALLS the function immediately and stores the RESULT
	{"map": extract_call_data(resource_key="calls")}
	
	# This stores the FUNCTION itself to be called later
	{"map": create_extract_call_data("calls")}
```
- Factory pattern solution
```python
def create_extract_call_data(resource_key):
    """Factory function - creates a customized data extractor"""
    
    def extract_call_data(record):  # Closure - captures resource_key
        """This function has the exact signature DLT expects"""
        # Your extraction logic here, using the captured resource_key
        pass
	    
    return extract_call_data  # Return the function, don't call it

# Usage in config
{"map": create_extract_call_data("calls")}  # Returns a function for DLT to use
```

### 2. Infinite Pagination:
- **HeaderLinkPaginator works correctly when there's actual data** - it properly stops when the last page with records omits `rel="next"`.
  **But when the API returns zero records from the very beginning**, Retreaver still generates `rel="next"` links on empty pages, causing infinite pagination.
- The solution was to create a custom `RetreaverPaginator` class that extends `HeaderLinkPaginator` but adds a check in `update_state()` to detect empty responses (`if not data`) and explicitly stops pagination by setting `self._has_next_page = False`. After registering this custom paginator with DLT using `register_paginator("retreaver_paginator", RetreaverPaginator)`, it can be used in the `RESTAPIConfig` dictionary via `"paginator": {"type": "retreaver_paginator"}`, which now properly handles both normal pagination (following Link headers when data exists) and edge cases (stopping when the API returns empty pages with misleading next links).
```Python
class RetreaverPaginator(HeaderLinkPaginator):

""" Stop pagination when response is empty,
Even if Link header has rel='next' """

	def update_state(self, response: Response, data: Any = None) -> None:
		super().update_state(response, data)
	
		# Stop if we got an empty response
		if not data:
			logger.info("Empty response received (len=0), stopping Retreaver pagination")
			self._has_next_page = False
	
	# Register the retreaver paginator
	register_paginator("retreaver_paginator", RetreaverPaginator)
```

