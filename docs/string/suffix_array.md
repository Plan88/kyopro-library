## Rust
```Rust
mod string {
    /// Build suffix array for a generic slice of `T: Ord`.
    /// Appends sentinel internally. Returns SA of length n+1 where sa[0] = n (sentinel).
    pub fn suffix_array<T: Ord>(s: &[T]) -> Vec<usize> {
        let n = s.len();
        if n == 0 {
            return vec![0];
        }

        // Coordinate compression: map elements to [1, distinct_count]
        let mut order: Vec<usize> = (0..n).collect();
        order.sort_by(|&a, &b| s[a].cmp(&s[b]));

        let mut t = vec![0usize; n + 1];
        let mut rank = 1usize;
        t[order[0]] = rank;
        for i in 1..n {
            if s[order[i]] != s[order[i - 1]] {
                rank += 1;
            }
            t[order[i]] = rank;
        }
        // t[n] = 0 is the sentinel (already 0)

        sa_is(&t, rank + 1)
    }

    /// SA-IS: O(N) Suffix Array construction algorithm
    /// (Nong, Zhang, Chan 2009)
    ///
    /// Input: integer slice `s` where each element is in [0, upper).
    /// Returns the suffix array (0-indexed).
    ///
    /// For string usage, append a sentinel (0) and set upper = max_char + 1.
    fn sa_is(s: &[usize], upper: usize) -> Vec<usize> {
        let n = s.len();
        if n == 0 {
            return vec![];
        }
        if n == 1 {
            return vec![0];
        }
        if n == 2 {
            return if s[0] < s[1] { vec![0, 1] } else { vec![1, 0] };
        }

        // Step 1: classify each suffix as S-type or L-type
        // is_s[i] = true means suffix i is S-type
        let mut is_s = vec![false; n];
        is_s[n - 1] = true; // last character is S-type (sentinel)
        for i in (0..n - 1).rev() {
            is_s[i] = if s[i] < s[i + 1] {
                true
            } else if s[i] > s[i + 1] {
                false
            } else {
                is_s[i + 1]
            };
        }

        // Step 2: find bucket boundaries
        let mut bucket = vec![0usize; upper];
        for &c in s {
            bucket[c] += 1;
        }
        // bucket_start[c] = start index of bucket c, bucket_end[c] = end index (exclusive)
        let mut bucket_start = vec![0usize; upper];
        let mut bucket_end = vec![0usize; upper];
        {
            let mut sum = 0;
            for c in 0..upper {
                bucket_start[c] = sum;
                sum += bucket[c];
                bucket_end[c] = sum;
            }
        }

        // Step 3: find all LMS positions
        let mut lms = Vec::new();
        for i in 1..n {
            if is_s[i] && !is_s[i - 1] {
                lms.push(i);
            }
        }

        // Induced sorting helper
        let induced_sort = |sa: &mut Vec<usize>, lms_order: &[usize]| {
            sa.fill(usize::MAX);

            // Place LMS suffixes at the end of their buckets (in reverse order of lms_order)
            let mut tail = bucket_end.clone();
            for &i in lms_order.iter().rev() {
                tail[s[i]] -= 1;
                sa[tail[s[i]]] = i;
            }

            // Induced sort L-type
            let mut head = bucket_start.clone();
            for j in 0..n {
                if sa[j] == usize::MAX || sa[j] == 0 {
                    continue;
                }
                let i = sa[j] - 1;
                if !is_s[i] {
                    sa[head[s[i]]] = i;
                    head[s[i]] += 1;
                }
            }

            // Induced sort S-type
            let mut tail = bucket_end.clone();
            for j in (0..n).rev() {
                if sa[j] == usize::MAX || sa[j] == 0 {
                    continue;
                }
                let i = sa[j] - 1;
                if is_s[i] {
                    tail[s[i]] -= 1;
                    sa[tail[s[i]]] = i;
                }
            }
        };

        // Step 4: initial induced sort with LMS in original order
        let mut sa = vec![usize::MAX; n];
        induced_sort(&mut sa, &lms);

        // Step 5: compact sorted LMS substrings and assign names
        // Determine the order of LMS substrings
        let sorted_lms: Vec<usize> = sa
            .iter()
            .copied()
            .filter(|&i| i != usize::MAX && i > 0 && is_s[i] && !is_s[i - 1])
            .collect();
        // Also include position 0 if it's LMS (it never is for n>=2, but be safe)
        if is_s[0] {
            // position 0 can't be LMS by definition (no i-1)
        }

        // Compare two LMS substrings for equality
        let lms_equal = |a: usize, b: usize| -> bool {
            // Compare characters until we hit the end of both LMS substrings
            for d in 0..n {
                let ca = s[a + d];
                let cb = s[b + d];
                let sa = is_s[a + d];
                let sb = is_s[b + d];
                if ca != cb || sa != sb {
                    return false;
                }
                // If d > 0 and both are LMS, we've reached the end of the LMS substring
                if d > 0 {
                    let lms_a = sa && (a + d == 0 || !is_s[a + d - 1]);
                    let lms_b = sb && (b + d == 0 || !is_s[b + d - 1]);
                    if lms_a || lms_b {
                        return lms_a && lms_b;
                    }
                }
            }
            false
        };

        let mut name = 0usize;
        let mut prev = usize::MAX;
        let mut lms_names = vec![usize::MAX; n]; // lms_names[i] = name for LMS position i
        for &pos in &sorted_lms {
            if prev == usize::MAX || !lms_equal(prev, pos) {
                name += 1;
            }
            lms_names[pos] = name - 1;
            prev = pos;
        }

        // Step 6: if names are not unique, recurse
        if name < lms.len() {
            // Build reduced string from LMS positions in text order
            let reduced: Vec<usize> = lms.iter().map(|&i| lms_names[i]).collect();
            let rec_sa = sa_is(&reduced, name);

            // Reorder LMS by the recursive SA result
            let sorted_lms_final: Vec<usize> = rec_sa.iter().map(|&i| lms[i]).collect();
            induced_sort(&mut sa, &sorted_lms_final);
        } else {
            // Names are unique; sorted_lms is already correct
            induced_sort(&mut sa, &sorted_lms);
        }

        sa
    }
}
```
