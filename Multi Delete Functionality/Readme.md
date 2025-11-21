Laravel Multi-Selection Delete Feature

This guide explains how to implement a multi-selection delete feature in a Laravel application. It includes Blade markup, JavaScript (converted to jQuery), route configuration, and controller logic.


---

📌 Features

Select multiple rows using checkboxes.

Delete selected records using a single "Delete Selected" button.

Confirmation prompt before deletion.

Activity logging and transaction-safe deletion.



---

🖥️ Blade File Implementation

Delete Selected Button

<a class="dropdown-item cursor-pointer" id="delete-selected">Delete Selected</a>

Checkbox Inside Table

<td>
    <input class="form-check-input row-checkbox" type="checkbox" name="ids[]" value="{{ $group->id }}">
</td>


---

🧩 jQuery Version of Delete Selected Script

$('#delete-selected').on('click', function () {
    let selected = [];

    $('.row-checkbox:checked').each(function () {
        selected.push($(this).val());
    });

    if (selected.length === 0) {
        alert('Please select at least one customer group.');
        return;
    }

    if (confirm('Are you sure you want to delete selected customer groups?')) {
        let form = $('<form>', {
            method: 'POST',
            action: '{{ route('delete.selected.customers.group') }}'
        });

        form.append('@csrf');
        form.append('<input type="hidden" name="_method" value="DELETE">');
        form.append(`<input type="hidden" name="ids" value="${selected.join(',')}">`);

        $('body').append(form);
        form.submit();
    }
});


---

🛣️ Route Definition

Route::delete('/customers-group/delete-selected', 'deleteSelected')->name('delete.selected.customers.group');


---

🧭 Controller Method: deleteSelected()

/**
 * Delete multiple customer groups
 */
public function deleteSelected(Request $request)
{
    $validator = Validator::make($request->all(), [
        'ids' => 'required|string',
    ]);

    if ($validator->fails()) {
        return redirect()->back()->with('error', 'No customer groups selected.');
    }

    DB::beginTransaction();

    try {
        $ids = is_array($request->ids) ? $request->ids : explode(',', $request->ids);

        if (empty($ids)) {
            return redirect()->back()->with('error', 'No customer groups selected.');
        }

        $deleted = CustomerGroup::destroy($ids);

        DB::commit();

        activity()
            ->causedBy(Auth::user())
            ->log('Deleted ' . $deleted . ' customer groups');

        if ($deleted > 0) {
            return redirect()->back()->with('success', "Successfully deleted {$deleted} customer group(s).");
        } else {
            return redirect()->back()->with('error', 'No groups deleted.');
        }
    } catch (\Exception $e) {
        DB::rollBack();
        Log::error('Error deleting customer groups: ' . $e->getMessage());

        return redirect()->back()->with('error', 'Error: ' . $e->getMessage());
    }
}


---

✅ Summary

This setup allows safe multi-row deletion.

Uses CSRF protection, DELETE method spoofing, and Laravel activity logs.

Ensures database safety with transactions.
