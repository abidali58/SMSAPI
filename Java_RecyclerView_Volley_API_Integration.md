# Java + RecyclerView + Volley API Integration

This guide walks you through integrating a REST API in an Android application using Java, displaying data in a `RecyclerView`, and fetching it using the `Volley` library.

---

## Step 1: Add Internet Permission in AndroidManifest.xml

To fetch data from an external API, Android applications need internet access permissions. This is critical because Android follows strict security models that disable network access by default. Add the following line in your `AndroidManifest.xml` file **outside the `<application>` tag** but inside the `<manifest>` tag:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Without this, network requests will fail, and you won’t be able to retrieve any data from your API.

---

## Step 2: Add Dependencies in `build.gradle`

Volley and RecyclerView are not built into the core SDK. Open your module-level `build.gradle` and add the following inside the `dependencies` block:

```gradle
implementation 'com.android.volley:volley:1.2.1'
implementation 'androidx.recyclerview:recyclerview:1.3.2'
```

After adding these, **sync the project** to ensure that the libraries are downloaded and integrated.

---

## Step 3: Create a Layout for RecyclerView Item

Every item in the RecyclerView needs a layout that defines how it appears. Create a new file `res/layout/item_job.xml`:

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:padding="10dp"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <TextView
        android:id="@+id/tvJobTitle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Job Title" />

    <TextView
        android:id="@+id/tvJobDetails"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Job Details" />
</LinearLayout>
```

---

## Step 4: Create a Java Model Class for Jobs

This class represents each job record returned by the API. It stores job ID, title, minimum salary, and maximum salary.

```java
public class Job {
    public String job_id;
    public String job_title;
    public String min_salary;
    public String max_salary;

    public Job(String job_id, String job_title, String min_salary, String max_salary) {
        this.job_id = job_id;
        this.job_title = job_title;
        this.min_salary = min_salary;
        this.max_salary = max_salary;
    }
}
```

---

## Step 5: Create a RecyclerView Adapter

The adapter bridges your data with the RecyclerView UI components. Create `JobAdapter.java`:

```java
public class JobAdapter extends RecyclerView.Adapter<JobAdapter.JobViewHolder> {
    private List<Job> jobList;

    public JobAdapter(List<Job> jobList) {
        this.jobList = jobList;
    }

    @NonNull
    @Override
    public JobViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_job, parent, false);
        return new JobViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull JobViewHolder holder, int position) {
        Job job = jobList.get(position);
        holder.tvJobTitle.setText(job.job_title);
        holder.tvJobDetails.setText("ID: " + job.job_id + " | Salary: " + job.min_salary + "-" + job.max_salary);
    }

    @Override
    public int getItemCount() {
        return jobList.size();
    }

    public static class JobViewHolder extends RecyclerView.ViewHolder {
        TextView tvJobTitle, tvJobDetails;

        public JobViewHolder(@NonNull View itemView) {
            super(itemView);
            tvJobTitle = itemView.findViewById(R.id.tvJobTitle);
            tvJobDetails = itemView.findViewById(R.id.tvJobDetails);
        }
    }
}
```

---

## Step 6: Add RecyclerView to `activity_main.xml`

```xml
<androidx.recyclerview.widget.RecyclerView
    android:id="@+id/jobRecyclerView"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

---

## Step 7: Use Volley to Fetch API Data

In your `MainActivity.java`, set up the RecyclerView and fetch API data using Volley.

```java
public class MainActivity extends AppCompatActivity {

    private RecyclerView jobRecyclerView;
    private List<Job> jobList;
    private JobAdapter jobAdapter;
    private final String apiUrl = "https://opulent-meme-97wgp4jpjgv6fgqv-5001.app.github.dev/jobs";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        jobRecyclerView = findViewById(R.id.jobRecyclerView);
        jobRecyclerView.setLayoutManager(new LinearLayoutManager(this));
        jobList = new ArrayList<>();
        jobAdapter = new JobAdapter(jobList);
        jobRecyclerView.setAdapter(jobAdapter);

        loadJobsFromApi();
    }

    private void loadJobsFromApi() {
        RequestQueue queue = Volley.newRequestQueue(this);

        StringRequest stringRequest = new StringRequest(Request.Method.GET, apiUrl,
            response -> {
                try {
                    JSONArray jsonArray = new JSONArray(response);
                    for (int i = 0; i < jsonArray.length(); i++) {
                        JSONObject obj = jsonArray.getJSONObject(i);
                        Job job = new Job(
                            obj.getString("job_id"),
                            obj.getString("job_title"),
                            obj.getString("min_salary"),
                            obj.getString("max_salary")
                        );
                        jobList.add(job);
                    }
                    jobAdapter.notifyDataSetChanged();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },
            error -> error.printStackTrace()
        );

        queue.add(stringRequest);
    }
}
```

---

## ✅ Output

You will now see a scrollable list of jobs retrieved from your API, displayed dynamically using RecyclerView and populated using the Volley networking library.

---

## Next Steps

- Add click listeners to open job detail pages.
- Integrate pagination or search.
- Cache data for offline use.

