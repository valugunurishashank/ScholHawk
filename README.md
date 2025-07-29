# ScholHawk Documentation

This document provides an overview of the backend system for **ScholHawk**, a platform designed to intelligently connect students with relevant scholarship opportunities. Built using Supabase and PostgreSQL, this document highlights key aspects of database design, intelligent matching algorithms, and robust security implementation.

### 1. Database Schema Design

At the core of ScholHawk are two primary, normalized tables: `user_profiles` for student information and `scholarships` for scholarship listings. For this project, the database schema was first defined, and tables with appropriate columns were created based on the metadata. A realistic dataset was then meticulously assembled, involving thorough research and compilation of 30 current scholarship listings into an Excel sheet, alongside 10 mock user profiles representing diverse student backgrounds, similarly organized in Excel. This prepared data was then imported into Supabase via CSV files.

This schema is designed for **normalization**, minimizing data redundancy and ensuring data integrity. Using `UUID`s for IDs provides global uniqueness. `TEXT[]` for `tags` allows flexible multi-value storage and efficient array operations. Categorical fields are `Text` for flexibility.

### 2. Matching Logic

The core of ScholHawk's intelligence lies in the PostgreSQL function: `get_matching_scholarships(p_user_id UUID)`. This function efficiently identifies and ranks relevant scholarships for a given student.

#### **How it Works:**

1.  **User Profile Retrieval:** The function first fetches the complete profile of the specified `p_user_id`.

2.  **Initial Filtering (`WHERE` clause):** Scholarships are filtered based on essential criteria. A scholarship must meet *all* these conditions to be considered:

    * **Active & Deadline:** Must be active and have a future or current deadline.

    * **GPA:** User's GPA must meet or exceed the scholarship's requirement.

    * **Field of Study:** Scholarship's field is "Any", or it partially matches the user's field (e.g., "STEM" matches ["Engineering", "STEM"]).

    * **Tags:** At least one tag must overlap, or the scholarship has no tags specified.

    * **Citizenship Status:** Scholarship status is "Not specified", or the user's status is found within the scholarship's (potentially comma-separated) list.

    * **Academic Level:** Scholarship level is "Any", "Not specified", or it partially matches the user's academic level.
### Bonus
### Relevance Scoring (`match_score`):
The `match_score` in the `get_matching_scholarships(p_user_id UUID)` function calculates a scholarship's relevance to a user by combining three weighted factors:

* **GPA Score (50% weight):** User's GPA, normalized and applied only if it meets the scholarship's requirement.

* **Field of Study Score (30% weight):** A fixed bonus if the scholarship's field is "Any" or matches the user's.

* **Tag Alignment Score (20% weight):** A bonus based on the proportion of overlapping tags between the user and the scholarship.

These weighted contributions are summed to give the final match_score, with higher scores indicating a better fit.

**Ordering:** Results are ordered by `match_score` descending, showing the best matches first.

### Row-Level Security (RLS) Implementation:

Appropriate Row-Level Security (RLS) policies are configured on sensitive tables (e.g., `user_profiles`, `scholarships`) to ensure that users can only access data they are authorized. RLS is active and correctly enforced to prevent unauthorized data exposure or manipulation. For example, users can only view their own user_profiles record.

### Assumptions and Future Ideas

#### **Assumptions:**
* **User Data Availability and Consistency:**
It is assumed that users have successfully completed a sign-up process where their relevant profile data (e.g., GPA, academic level, citizenship, etc.) is collected and stored in the `user_profiles`. This user profile data is assumed to be consistent and properly formatted (e.g., string fields adhere to expected patterns, categorical values are standardized).

* **Admin Authentication and Data Management:**
It is assumed that administrative users are managed through a dedicated `admin_users`, which stores all relevant administrator information. Configured to grant administrators comprehensive permissions (SELECT, INSERT, UPDATE, DELETE) to `scholarship` and `user_profile`.


#### **Future Enhancements:**

* **Advanced Text Matching:** Matching based on descriptions or fields for broader relevance.

* **User Dashboard & Notifications:** Develop a frontend for users to view matches, save favorites, and receive alerts for new opportunities.

* **Admin Management:** Build a dedicated admin interface for easier scholarship and user data management.

* **Geolocation:** Incorporate location data to allow students to find scholarships specific to their region.

* **Performance Optimization:** Implement advanced indexing and potentially partitioning for very large datasets.
