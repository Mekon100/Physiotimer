import streamlit as st
import time
import math
from typing import List, Tuple
import threading

# --- Text-to-Speech (TTS) Configuration ---
# Streamlit runs on a server, so direct server-side TTS might not be
# practical or desirable for client-side audio. We'll rely on printing
# announcements to the Streamlit app's output area.
# If you need client-side audio, you would typically use JavaScript
# in a web application, which is outside the scope of a pure Streamlit app.

def speak(text: str):
    """
    This function prints verbal announcements to the Streamlit app.
    Actual voice output is not directly supported in standard Streamlit.
    """
    st.info(f"ANNOUNCEMENT: {text}") # Use st.info to highlight announcements

# --- Data Structures and Helper Functions ---

class Exam:
    """Represents a practical assessment examination."""
    def __init__(self, base_assessment_time_minutes: float, reading_time_minutes: float):
        if base_assessment_time_minutes < 0 or reading_time_minutes < 0:
            raise ValueError("Time values cannot be negative.")
        self.base_assessment_time = base_assessment_time_minutes
        self.reading_time = reading_time_minutes

    def get_total_standard_time(self) -> float:
        """Returns the total time for a standard student (base + reading)."""
        return self.base_assessment_time + self.reading_time

def calculate_group_time(exam: Exam, additional_needs_percentage: float) -> float:
    """
    Calculates the total time for a student group based on exam details
    and their additional needs percentage.
    The additional percentage is applied to the COMBINED base assessment time
    and reading time.
    """
    standard_total_time = exam.get_total_standard_time()
    if additional_needs_percentage > 0:
        # Apply percentage to the total standard time (base + reading)
        extension_factor = 1 + (additional_needs_percentage / 100.0)
        return standard_total_time * extension_factor
    else:
        # Standard time is base + reading
        return standard_total_time

def format_time_hh_mm_ss(total_seconds: int) -> str:
    """Formats total seconds into HH:MM:SS string."""
    hours = total_seconds // 3600
    minutes = (total_seconds % 3600) // 60
    secs = total_seconds % 60
    return f"{hours:02d}:{minutes:02d}:{secs:02d}"

# --- Timer and Announcement Logic ---

# We need to manage the timer state using Streamlit's session state
# and potentially use threading if we want the timer to update independently
# of user interaction (though Streamlit reruns the script on interaction).
# A simpler approach in Streamlit is to use a loop and st.empty() to update
# the display, which will block the script execution while the timer runs.

def run_countdown_timer_streamlit(total_duration_minutes: float):
    """
    Runs a real-time countdown timer within the Streamlit app.
    Announcements are printed to the app.
    This function blocks execution while the timer is running.
    """
    total_seconds = int(total_duration_minutes * 60)

    # Define key announcement points in minutes (e.g., 60, 50, 40, 30, 20, 10, 5, 1)
    announcement_minutes_set = set()

    for m in range(int(math.ceil(total_duration_minutes)), 0, -1):
         if m % 10 == 0 or m in [60, 30, 20, 15, 10, 5, 4, 3, 2, 1]:
             announcement_minutes_set.add(m)

    for m in [30, 25, 20, 15, 10, 5, 4, 3, 2, 1]:
        if m <= total_duration_minutes:
            announcement_minutes_set.add(m)

    announcement_minutes_list = sorted([m for m in announcement_minutes_set if m > 0], reverse=True)

    announcement_seconds_at_end = {30, 10, 5}

    st.subheader("Countdown Timer")
    timer_placeholder = st.empty() # Placeholder to update the timer display
    announcement_placeholder = st.empty() # Placeholder for announcements

    announcement_placeholder.info(f"Starting Timer for {total_duration_minutes:.2f} minutes ({format_time_hh_mm_ss(total_seconds)})")
    speak(f"The assessment has started. You have {math.ceil(total_duration_minutes)} minutes remaining.")

    start_time = time.time()

    last_announced_minute = -1
    last_announced_second = -1
    next_minute_announcement_index = 0

    while True:
        elapsed_seconds = int(time.time() - start_time)
        remaining_seconds = total_seconds - elapsed_seconds

        display_remaining_seconds = max(0, remaining_seconds)
        timer_placeholder.metric("Remaining Time", format_time_hh_mm_ss(display_remaining_seconds))

        if remaining_seconds <= 0:
            announcement_placeholder.info("Time's up! Please stop writing.")
            speak("Time's up! Please stop writing.")
            break

        current_remaining_minutes = math.ceil(remaining_seconds / 60)

        if next_minute_announcement_index < len(announcement_minutes_list):
            expected_announcement_minute = announcement_minutes_list[next_minute_announcement_index]
            if current_remaining_minutes <= expected_announcement_minute and current_remaining_minutes != last_announced_minute:
                 speak(f"{expected_announcement_minute} minutes remaining.")
                 last_announced_minute = current_remaining_minutes
                 next_minute_announcement_index += 1

        if remaining_seconds in announcement_seconds_at_end and remaining_seconds != last_announced_second:
            speak(f"{remaining_seconds} seconds remaining.")
            last_announced_second = remaining_seconds
            announcement_seconds_at_end.discard(remaining_seconds)

        time.sleep(1) # Sleep for 1 second

# --- Streamlit App Layout ---

def main_streamlit():
    """Main function to run the Streamlit application."""
    st.set_page_config(page_title="Assessment Time Manager", layout="centered")

    st.title("Practical Assessment Time Manager")

    st.markdown("""
        This application helps manage timekeeping for practical assessments,
        including accommodating additional time needs for student groups.
    """)

    # --- Set Up Exam Times ---
    st.header("Set Up Exam Times")
    base_time = st.number_input("Enter the base assessment time required (in minutes):", min_value=0.0, value=20.0, step=0.5)
    reading_time = st.number_input("Enter the reading time (in minutes):", min_value=0.0, value=1.0, step=0.5)

    try:
        exam = Exam(base_time, reading_time)
        st.success(f"Exam set: Base Assessment Time = {exam.base_assessment_time:.2f} mins, Reading Time = {exam.reading_time:.2f} mins.")
        st.info(f"Standard total time (base + reading): {exam.get_total_standard_time():.2f} minutes.")
    except ValueError as e:
        st.error(f"Error setting up exam: {e}")
        return # Stop execution if exam setup fails

    # --- Add Student Groups ---
    st.header("Add Student Groups")

    # Use session state to store student groups
    if 'student_groups' not in st.session_state:
        st.session_state.student_groups = []

    num_standard_students = st.number_input("Enter the number of students with standard time:", min_value=0, value=0, step=1)
    if num_standard_students > 0 and (0, 0.0) not in st.session_state.student_groups:
         # Add standard group only if it doesn't exist and count is > 0
         # This prevents duplicates on rerun
         st.session_state.student_groups.append((num_standard_students, 0.0))
         # Simple way to trigger a rerun after adding the standard group
         st.experimental_rerun()


    st.subheader("Add Groups with Additional Needs")
    with st.form("additional_group_form", clear_on_submit=True):
        col1, col2 = st.columns(2)
        with col1:
            num_students_in_group = st.number_input("Number of students in this group:", min_value=0, step=1, key="num_students_input")
        with col2:
            additional_percentage = st.number_input("Additional time percentage (e.g., 25 for 25%):", min_value=0.0, max_value=500.0, step=0.5, key="percentage_input")

        add_group_button = st.form_submit_button("Add Group")

        if add_group_button:
            if num_students_in_group > 0:
                st.session_state.student_groups.append((num_students_in_group, additional_percentage))
                st.success(f"Added group: {num_students_in_group} students with {additional_percentage:.0f}% extra time.")
                # Streamlit reruns on form submission, so no explicit rerun needed here
            elif num_students_in_group == 0 and additional_percentage > 0:
                 st.warning("Number of students must be greater than 0 for an additional needs group.")


    # Display current groups
    if st.session_state.student_groups:
        st.subheader("Current Student Groups")
        groups_df = st.session_state.student_groups # Use the list directly
        
        # Create a display table/markdown
        st.markdown("---")
        st.markdown(f"{'Students Count':<15} | {'Additional %':<12}")
        st.markdown("---")
        for count, percentage in groups_df:
             percentage_display = f"{percentage:.0f}%" if percentage > 0 else "Standard"
             st.markdown(f"{count:<15} | {percentage_display:<12}")
        st.markdown("---")

        if st.button("Clear All Groups"):
             st.session_state.student_groups = []
             st.experimental_rerun() # Rerun to clear the display


    # 3. Calculate and Display Distinct Total Times
    st.header("Calculated Assessment Times for Groups")

    if st.session_state.student_groups:
        calculated_times_info = {} # {total_time_minutes: percentage_label}
        all_distinct_times_for_timer = []

        st.markdown(f"{'Students Count':<15} | {'Additional %':<12} | {'Total Time (mins)':<18} | {'Total Time (HH:MM:SS)':<20}")
        st.markdown("-" * 75)

        for count, percentage in st.session_state.student_groups:
            total_time = calculate_group_time(exam, percentage)
            time_str_hh_mm_ss = format_time_hh_mm_ss(int(total_time * 60))
            percentage_display = f"{percentage:.0f}%" if percentage > 0 else "Standard"

            st.markdown(f"{count:<15} | {percentage_display:<12} | {total_time:<18.2f} | {time_str_hh_mm_ss:<20}")

            if total_time not in calculated_times_info:
                calculated_times_info[total_time] = percentage_display
                all_distinct_times_for_timer.append(total_time)

        st.markdown("---")

        if all_distinct_times_for_timer:
            st.header("Choose Timer Duration")

            # Sort distinct times for clear presentation
            all_distinct_times_for_timer.sort()

            # Create a list of display options for the selectbox
            timer_options = [
                f"{duration:.2f} minutes ({calculated_times_info[duration]}) ({format_time_hh_mm_ss(int(duration * 60))})"
                for duration in all_distinct_times_for_timer
            ]

            selected_option = st.selectbox(
                "Select the time duration to run the timer for:",
                options=timer_options
            )

            # Extract the duration from the selected option string
            selected_duration_str = selected_option.split(" minutes")[0]
            selected_duration = float(selected_duration_str)

            if st.button("Start Timer"):
                 # Run the timer function
                 run_countdown_timer_streamlit(selected_duration)

    else:
        st.info("Add student groups to calculate times.")


if __name__ == "__main__":
    main_streamlit()
