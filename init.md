## 테스트 실패 정보

아래는 실패한 테스트와 테스트의 에러메시지를 담은 pytest 출력입니다. 실패한 테스트 함수는 [여기](./repo/pandas/tests/dtypes/test_missing.py#L363)서도 보실 수 있습니다.

```python
============================= test session starts ==============================
platform linux -- Python 3.8.3, pytest-5.4.3, py-1.8.1, pluggy-0.13.1
rootdir: /home/user/BugsInPy/temp/projects/pandas, inifile: setup.cfg
plugins: hypothesis-5.16.0
collected 1 item

pandas/tests/dtypes/test_missing.py F                                    [100%]

=================================== FAILURES ===================================
_________________________ test_array_equivalent_nested _________________________

    def test_array_equivalent_nested():
        # reached in groupby aggregations, make sure we use np.any when checking
        #  if the comparison is truthy
        left = np.array([np.array([50, 70, 90]), np.array([20, 30, 40])], dtype=object)
        right = np.array([np.array([50, 70, 90]), np.array([20, 30, 40])], dtype=object)
    
>       assert array_equivalent(left, right, strict_nan=True)

pandas/tests/dtypes/test_missing.py:369: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

left = array([[50, 70, 90],
       [20, 30, 40]], dtype=object)
right = array([[50, 70, 90],
       [20, 30, 40]], dtype=object)
strict_nan = True

    def array_equivalent(left, right, strict_nan=False):
        """
        True if two arrays, left and right, have equal non-NaN elements, and NaNs
        in corresponding locations.  False otherwise. It is assumed that left and
        right are NumPy arrays of the same dtype. The behavior of this function
        (particularly with respect to NaNs) is not defined if the dtypes are
        different.
    
        Parameters
        ----------
        left, right : ndarrays
        strict_nan : bool, default False
            If True, consider NaN and None to be different.
    
        Returns
        -------
        b : bool
            Returns True if the arrays are equivalent.
    
        Examples
        --------
        >>> array_equivalent(
        ...     np.array([1, 2, np.nan]),
        ...     np.array([1, 2, np.nan]))
        True
        >>> array_equivalent(
        ...     np.array([1, np.nan, 2]),
        ...     np.array([1, 2, np.nan]))
        False
        """
    
        left, right = np.asarray(left), np.asarray(right)
    
        # shape compat
        if left.shape != right.shape:
            return False
    
        # Object arrays can contain None, NaN and NaT.
        # string dtypes must be come to this path for NumPy 1.7.1 compat
        if is_string_dtype(left) or is_string_dtype(right):
    
            if not strict_nan:
                # isna considers NaN and None to be equivalent.
                return lib.array_equivalent_object(
                    ensure_object(left.ravel()), ensure_object(right.ravel())
                )
    
            for left_value, right_value in zip(left, right):
                if left_value is NaT and right_value is not NaT:
                    return False
    
                elif isinstance(left_value, float) and np.isnan(left_value):
                    if not isinstance(right_value, float) or not np.isnan(right_value):
                        return False
                else:
>                   if left_value != right_value:
E                   ValueError: The truth value of an array with more than one element is ambiguous. Use a.any() or a.all()

pandas/core/dtypes/missing.py:448: ValueError
```